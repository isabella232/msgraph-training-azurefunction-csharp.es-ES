---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822735"
---
<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio finalizará la implementación de la función de Azure `GetMyNewestMessage` y se actualizará el cliente de prueba para llamar a la función.

La función de Azure usa el [flujo "en nombre de"](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). El orden básico de los eventos en este flujo es:

- La aplicación de prueba usa un flujo de autenticación interactivo para permitir que el usuario inicie sesión y conceda el consentimiento. Devuelve un token que se encuentra en el ámbito de la función de Azure. El token **no** contiene ningún ámbito de Microsoft Graph.
- La aplicación de prueba invoca la función de Azure y envía su token de acceso en el `Authorization` encabezado.
- La función de Azure valida el token y, a continuación, intercambia ese token para un segundo token de acceso que contiene los ámbitos de Microsoft Graph.
- La función de Azure llama a Microsoft Graph en nombre del usuario con el segundo token de acceso.

> [!IMPORTANT]
> Para evitar almacenar el identificador de aplicación y el secreto en el origen, deberá usar el [Administrador de secretos de .net](https://docs.microsoft.com/aspnet/core/security/app-secrets) para almacenar estos valores. El administrador de secretos solo se usa con fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.

## <a name="add-authentication-to-the-single-page-application"></a>Agregar autenticación a la aplicación de una sola página

Comience por agregar la autenticación al SPA. Esto permitirá que la aplicación obtenga un token de acceso que conceda acceso para llamar a la función de Azure. Dado que se trata de un SPA, usará el [flujo de código de autorización con PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).

1. Cree un nuevo archivo en el directorio **TestClient** denominado **config.js** y agregue el código siguiente.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Reemplace `YOUR_TEST_APP_APP_ID_HERE` por el identificador de aplicación que creó en el portal de Azure para la aplicación de prueba de la **función de Microsoft Graph**. Reemplace `YOUR_TENANT_ID_HERE` por el valor de **identificador de directorio (tenant)** que copió desde el portal de Azure. Reemplace `YOUR_AZURE_FUNCTION_APP_ID_HERE` por el identificador de aplicación de la **función Graph de Azure**.

    > [!IMPORTANT]
    > Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo de **config.js** del control de código fuente para evitar la pérdida inadvertida de los identificadores de aplicación y el identificador de inquilino.

1. Cree un nuevo archivo en el directorio **TestClient** denominado **auth.js** y agregue el código siguiente.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Considere lo que hace este código.

    - Inicializa una `PublicClientApplication` con los valores almacenados en **config.js**.
    - Usa `loginPopup` para iniciar sesión del usuario, usando el ámbito de permiso para la función de Azure.
    - Almacena el nombre de usuario del usuario en la sesión.

    > [!IMPORTANT]
    > Como la aplicación usa `loginPopup` , es posible que debas cambiar el bloqueador de ventanas emergentes del explorador para permitir los elementos emergentes de `http://localhost:8080` .

1. Actualice la página e inicie sesión. La página debe actualizarse con el nombre de usuario, lo que indica que el inicio de sesión se realizó correctamente.

> [!TIP]
> Puede analizar el token de acceso en [https://jwt.ms](https://jwt.ms) y confirmar que la `aud` notificación es el identificador de la aplicación para la función de Azure y que la `scp` notificación contiene el ámbito de permisos de la función de Azure, no Microsoft Graph.

## <a name="add-authentication-to-the-azure-function"></a>Agregar autenticación a la función de Azure

En esta sección, implementará el flujo "en nombre de" en la `GetMyNewestMessage` función de Azure para obtener un token de acceso compatible con Microsoft Graph.

1. Inicialice el almacén secreto de desarrollo .NET abriendo la CLI en el directorio que contiene **GraphTutorial. csproj** y ejecutando el siguiente comando.

    ```Shell
    dotnet user-secrets init
    ```

1. Agregue el identificador de la aplicación, el secreto y el identificador de inquilino al almacén secreto mediante los comandos siguientes. Reemplace `YOUR_API_FUNCTION_APP_ID_HERE` por el identificador de aplicación de la **función Graph de Azure**. Reemplace `YOUR_API_FUNCTION_APP_SECRET_HERE` por el secreto de aplicación que ha creado en Azure portal para la **función Graph Azure**. Reemplace `YOUR_TENANT_ID_HERE` por el valor de **identificador de directorio (tenant)** que copió desde el portal de Azure.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Procesar el token del portador de entrada

En esta sección, implementará una clase para validar y procesar el token de portador enviado desde el SPA a la función de Azure.

1. Cree un nuevo directorio en el directorio **GraphTutorial** denominado **Authentication (autenticación** ).

1. Cree un nuevo archivo denominado **TokenValidationResult.CS** en la carpeta **./GraphTutorial/Authentication** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Cree un nuevo archivo denominado **TokenValidation.CS** en la carpeta **./GraphTutorial/Authentication** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Considere lo que hace este código.

- Asegúrese de que hay un token de portador en el `Authorization` encabezado.
- Comprueba la firma y el emisor desde la configuración de OpenID publicada de Azure.
- Comprueba que la audiencia ( `aud` Claim) coincide con el identificador de la aplicación de la función de Azure.
- Analiza el token y genera un identificador de cuenta de MSAL, que será necesario para aprovechar el almacenamiento en caché de tokens.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Crear un proveedor de autenticación en nombre de

1. Cree un nuevo archivo en el directorio de **autenticación** denominado **OnBehalfOfAuthProvider.CS** y agregue el siguiente código a ese archivo.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Tómese un momento para considerar lo que hace el código en **OnBehalfOfAuthProvider.CS** .

- En la `GetAccessToken` función, primero intenta obtener un token de usuario de la caché de tokens mediante `AcquireTokenSilent` . Si se produce un error, se usa el token de portador enviado por la aplicación de prueba a la función de Azure para generar una aserción de usuario. A continuación, se usa la aserción del usuario para obtener un token compatible con Graph mediante `AcquireTokenOnBehalfOf` .
- Implementa la `Microsoft.Graph.IAuthenticationProvider` interfaz, lo que permite pasar esta clase en el constructor de `GraphServiceClient` para autenticar las solicitudes salientes.

### <a name="implement-a-graph-client-service"></a>Implementar un servicio de cliente de Graph

En esta sección, implementará un servicio que se puede registrar para la [inserción de dependencias](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection). El servicio se usará para obtener un cliente de gráfico autenticado.

1. Cree un nuevo directorio en el directorio **GraphTutorial** denominado **servicios**.

1. Cree un nuevo archivo en el directorio de **servicios** denominado **IGraphClientService.CS** y agregue el siguiente código a ese archivo.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Cree un nuevo archivo en el directorio de **servicios** denominado **GraphClientService.CS** y agregue el siguiente código a ese archivo.

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. Agregue las siguientes propiedades a la `GraphClientService` clase.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Agregue las siguientes funciones a la `GraphClientService` clase.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Agregue una implementación de marcador de posición para la `GetAppGraphClient` función. Lo implementará en secciones posteriores.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    La `GetUserGraphClient` función toma los resultados de la validación de tokens y crea una autenticación `GraphServiceClient` para el usuario.

1. Cree un archivo nuevo en el directorio **GraphTutorial** denominado **Startup.CS** y agregue el código siguiente al archivo.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    Este código permitirá la [inserción de dependencias](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) en las funciones de Azure, exponiendo el `IConfiguration` objeto y el `GraphClientService` servicio.

### <a name="implement-getmynewestmessage-function"></a>Implementar la función GetMyNewestMessage

1. Abra **./GraphTutorial/GetMyNewestMessage.CS** y reemplace todo el contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Revise el código en GetMyNewestMessage.cs

Tómese un momento para considerar lo que hace el código en **GetMyNewestMessage.CS** .

- En el constructor, guarda el `IConfiguration` objeto que se pasa a través de la inserción de dependencias.
- En la `Run` función, hace lo siguiente:
  - Valida los valores de configuración necesarios que están presentes en el `IConfiguration` objeto.
  - Valida el token del portador y devuelve un `401` código de estado si el token no es válido.
  - Obtiene un cliente de Graph del `GraphClientService` para el usuario que realizó esta solicitud.
  - Usa el SDK de Microsoft Graph para obtener el mensaje más reciente de la bandeja de entrada del usuario y lo devuelve como un cuerpo JSON en la respuesta.

## <a name="call-the-azure-function-from-the-test-app"></a>Llamar a la función de Azure desde la aplicación de prueba

1. Abra **auth.js** y agregue la siguiente función para obtener un token de acceso.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Considere lo que hace este código.

    - En primer lugar, intenta obtener un token de acceso en modo silencioso, sin interacción del usuario. Como el usuario ya debe haber iniciado sesión, MSAL debe tener tokens para el usuario en su caché.
    - Si se produce un error que indica que el usuario tiene que interactuar, intenta obtener un token de forma interactiva.

1. Cree un nuevo archivo en el directorio **TestClient** denominado **azurefunctions.js** y agregue el código siguiente.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Cambie el directorio actual de la CLI al directorio **./GraphTutorial** y ejecute el siguiente comando para iniciar la función de Azure de forma local.

    ```Shell
    func start
    ```

1. Si aún no atiende el SPA, abra una segunda ventana de CLI y cambie el directorio actual al directorio **./TestClient** . Ejecute el siguiente comando para ejecutar la aplicación de prueba.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Abra el explorador y vaya a `http://localhost:8080`. Inicie sesión y seleccione el **último** elemento de navegación de mensajes. La aplicación muestra información acerca del mensaje más reciente en la bandeja de entrada del usuario.
