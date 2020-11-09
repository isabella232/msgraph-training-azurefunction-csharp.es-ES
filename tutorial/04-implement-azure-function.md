---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822735"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="142c2-101">En este ejercicio finalizará la implementación de la función de Azure `GetMyNewestMessage` y se actualizará el cliente de prueba para llamar a la función.</span><span class="sxs-lookup"><span data-stu-id="142c2-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="142c2-102">La función de Azure usa el [flujo "en nombre de"](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span><span class="sxs-lookup"><span data-stu-id="142c2-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="142c2-103">El orden básico de los eventos en este flujo es:</span><span class="sxs-lookup"><span data-stu-id="142c2-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="142c2-104">La aplicación de prueba usa un flujo de autenticación interactivo para permitir que el usuario inicie sesión y conceda el consentimiento.</span><span class="sxs-lookup"><span data-stu-id="142c2-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="142c2-105">Devuelve un token que se encuentra en el ámbito de la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="142c2-106">El token **no** contiene ningún ámbito de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="142c2-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="142c2-107">La aplicación de prueba invoca la función de Azure y envía su token de acceso en el `Authorization` encabezado.</span><span class="sxs-lookup"><span data-stu-id="142c2-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="142c2-108">La función de Azure valida el token y, a continuación, intercambia ese token para un segundo token de acceso que contiene los ámbitos de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="142c2-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="142c2-109">La función de Azure llama a Microsoft Graph en nombre del usuario con el segundo token de acceso.</span><span class="sxs-lookup"><span data-stu-id="142c2-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="142c2-110">Para evitar almacenar el identificador de aplicación y el secreto en el origen, deberá usar el [Administrador de secretos de .net](https://docs.microsoft.com/aspnet/core/security/app-secrets) para almacenar estos valores.</span><span class="sxs-lookup"><span data-stu-id="142c2-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="142c2-111">El administrador de secretos solo se usa con fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.</span><span class="sxs-lookup"><span data-stu-id="142c2-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="142c2-112">Agregar autenticación a la aplicación de una sola página</span><span class="sxs-lookup"><span data-stu-id="142c2-112">Add authentication to the single page application</span></span>

<span data-ttu-id="142c2-113">Comience por agregar la autenticación al SPA.</span><span class="sxs-lookup"><span data-stu-id="142c2-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="142c2-114">Esto permitirá que la aplicación obtenga un token de acceso que conceda acceso para llamar a la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="142c2-115">Dado que se trata de un SPA, usará el [flujo de código de autorización con PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span><span class="sxs-lookup"><span data-stu-id="142c2-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="142c2-116">Cree un nuevo archivo en el directorio **TestClient** denominado **config.js** y agregue el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="142c2-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="142c2-117">Reemplace `YOUR_TEST_APP_APP_ID_HERE` por el identificador de aplicación que creó en el portal de Azure para la aplicación de prueba de la **función de Microsoft Graph**.</span><span class="sxs-lookup"><span data-stu-id="142c2-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="142c2-118">Reemplace `YOUR_TENANT_ID_HERE` por el valor de **identificador de directorio (tenant)** que copió desde el portal de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="142c2-119">Reemplace `YOUR_AZURE_FUNCTION_APP_ID_HERE` por el identificador de aplicación de la **función Graph de Azure**.</span><span class="sxs-lookup"><span data-stu-id="142c2-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="142c2-120">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo de **config.js** del control de código fuente para evitar la pérdida inadvertida de los identificadores de aplicación y el identificador de inquilino.</span><span class="sxs-lookup"><span data-stu-id="142c2-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="142c2-121">Cree un nuevo archivo en el directorio **TestClient** denominado **auth.js** y agregue el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="142c2-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="142c2-122">Considere lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="142c2-122">Consider what this code does.</span></span>

    - <span data-ttu-id="142c2-123">Inicializa una `PublicClientApplication` con los valores almacenados en **config.js**.</span><span class="sxs-lookup"><span data-stu-id="142c2-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="142c2-124">Usa `loginPopup` para iniciar sesión del usuario, usando el ámbito de permiso para la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="142c2-125">Almacena el nombre de usuario del usuario en la sesión.</span><span class="sxs-lookup"><span data-stu-id="142c2-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="142c2-126">Como la aplicación usa `loginPopup` , es posible que debas cambiar el bloqueador de ventanas emergentes del explorador para permitir los elementos emergentes de `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="142c2-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="142c2-127">Actualice la página e inicie sesión.</span><span class="sxs-lookup"><span data-stu-id="142c2-127">Refresh the page and sign in.</span></span> <span data-ttu-id="142c2-128">La página debe actualizarse con el nombre de usuario, lo que indica que el inicio de sesión se realizó correctamente.</span><span class="sxs-lookup"><span data-stu-id="142c2-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

> [!TIP]
> <span data-ttu-id="142c2-129">Puede analizar el token de acceso en [https://jwt.ms](https://jwt.ms) y confirmar que la `aud` notificación es el identificador de la aplicación para la función de Azure y que la `scp` notificación contiene el ámbito de permisos de la función de Azure, no Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="142c2-129">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="142c2-130">Agregar autenticación a la función de Azure</span><span class="sxs-lookup"><span data-stu-id="142c2-130">Add authentication to the Azure Function</span></span>

<span data-ttu-id="142c2-131">En esta sección, implementará el flujo "en nombre de" en la `GetMyNewestMessage` función de Azure para obtener un token de acceso compatible con Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="142c2-131">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="142c2-132">Inicialice el almacén secreto de desarrollo .NET abriendo la CLI en el directorio que contiene **GraphTutorial. csproj** y ejecutando el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="142c2-132">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="142c2-133">Agregue el identificador de la aplicación, el secreto y el identificador de inquilino al almacén secreto mediante los comandos siguientes.</span><span class="sxs-lookup"><span data-stu-id="142c2-133">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="142c2-134">Reemplace `YOUR_API_FUNCTION_APP_ID_HERE` por el identificador de aplicación de la **función Graph de Azure**.</span><span class="sxs-lookup"><span data-stu-id="142c2-134">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="142c2-135">Reemplace `YOUR_API_FUNCTION_APP_SECRET_HERE` por el secreto de aplicación que ha creado en Azure portal para la **función Graph Azure**.</span><span class="sxs-lookup"><span data-stu-id="142c2-135">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="142c2-136">Reemplace `YOUR_TENANT_ID_HERE` por el valor de **identificador de directorio (tenant)** que copió desde el portal de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-136">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="142c2-137">Procesar el token del portador de entrada</span><span class="sxs-lookup"><span data-stu-id="142c2-137">Process the incoming bearer token</span></span>

<span data-ttu-id="142c2-138">En esta sección, implementará una clase para validar y procesar el token de portador enviado desde el SPA a la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-138">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="142c2-139">Cree un nuevo directorio en el directorio **GraphTutorial** denominado **Authentication (autenticación** ).</span><span class="sxs-lookup"><span data-stu-id="142c2-139">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="142c2-140">Cree un nuevo archivo denominado **TokenValidationResult.CS** en la carpeta **./GraphTutorial/Authentication** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="142c2-140">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="142c2-141">Cree un nuevo archivo denominado **TokenValidation.CS** en la carpeta **./GraphTutorial/Authentication** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="142c2-141">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="142c2-142">Considere lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="142c2-142">Consider what this code does.</span></span>

- <span data-ttu-id="142c2-143">Asegúrese de que hay un token de portador en el `Authorization` encabezado.</span><span class="sxs-lookup"><span data-stu-id="142c2-143">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="142c2-144">Comprueba la firma y el emisor desde la configuración de OpenID publicada de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-144">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="142c2-145">Comprueba que la audiencia ( `aud` Claim) coincide con el identificador de la aplicación de la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="142c2-145">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="142c2-146">Analiza el token y genera un identificador de cuenta de MSAL, que será necesario para aprovechar el almacenamiento en caché de tokens.</span><span class="sxs-lookup"><span data-stu-id="142c2-146">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="142c2-147">Crear un proveedor de autenticación en nombre de</span><span class="sxs-lookup"><span data-stu-id="142c2-147">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="142c2-148">Cree un nuevo archivo en el directorio de **autenticación** denominado **OnBehalfOfAuthProvider.CS** y agregue el siguiente código a ese archivo.</span><span class="sxs-lookup"><span data-stu-id="142c2-148">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="142c2-149">Tómese un momento para considerar lo que hace el código en **OnBehalfOfAuthProvider.CS** .</span><span class="sxs-lookup"><span data-stu-id="142c2-149">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="142c2-150">En la `GetAccessToken` función, primero intenta obtener un token de usuario de la caché de tokens mediante `AcquireTokenSilent` .</span><span class="sxs-lookup"><span data-stu-id="142c2-150">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="142c2-151">Si se produce un error, se usa el token de portador enviado por la aplicación de prueba a la función de Azure para generar una aserción de usuario.</span><span class="sxs-lookup"><span data-stu-id="142c2-151">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="142c2-152">A continuación, se usa la aserción del usuario para obtener un token compatible con Graph mediante `AcquireTokenOnBehalfOf` .</span><span class="sxs-lookup"><span data-stu-id="142c2-152">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="142c2-153">Implementa la `Microsoft.Graph.IAuthenticationProvider` interfaz, lo que permite pasar esta clase en el constructor de `GraphServiceClient` para autenticar las solicitudes salientes.</span><span class="sxs-lookup"><span data-stu-id="142c2-153">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="142c2-154">Implementar un servicio de cliente de Graph</span><span class="sxs-lookup"><span data-stu-id="142c2-154">Implement a Graph client service</span></span>

<span data-ttu-id="142c2-155">En esta sección, implementará un servicio que se puede registrar para la [inserción de dependencias](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="142c2-155">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="142c2-156">El servicio se usará para obtener un cliente de gráfico autenticado.</span><span class="sxs-lookup"><span data-stu-id="142c2-156">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="142c2-157">Cree un nuevo directorio en el directorio **GraphTutorial** denominado **servicios**.</span><span class="sxs-lookup"><span data-stu-id="142c2-157">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="142c2-158">Cree un nuevo archivo en el directorio de **servicios** denominado **IGraphClientService.CS** y agregue el siguiente código a ese archivo.</span><span class="sxs-lookup"><span data-stu-id="142c2-158">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="142c2-159">Cree un nuevo archivo en el directorio de **servicios** denominado **GraphClientService.CS** y agregue el siguiente código a ese archivo.</span><span class="sxs-lookup"><span data-stu-id="142c2-159">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="142c2-160">Agregue las siguientes propiedades a la `GraphClientService` clase.</span><span class="sxs-lookup"><span data-stu-id="142c2-160">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="142c2-161">Agregue las siguientes funciones a la `GraphClientService` clase.</span><span class="sxs-lookup"><span data-stu-id="142c2-161">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="142c2-162">Agregue una implementación de marcador de posición para la `GetAppGraphClient` función.</span><span class="sxs-lookup"><span data-stu-id="142c2-162">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="142c2-163">Lo implementará en secciones posteriores.</span><span class="sxs-lookup"><span data-stu-id="142c2-163">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="142c2-164">La `GetUserGraphClient` función toma los resultados de la validación de tokens y crea una autenticación `GraphServiceClient` para el usuario.</span><span class="sxs-lookup"><span data-stu-id="142c2-164">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="142c2-165">Cree un archivo nuevo en el directorio **GraphTutorial** denominado **Startup.CS** y agregue el código siguiente al archivo.</span><span class="sxs-lookup"><span data-stu-id="142c2-165">Create a new file in the **GraphTutorial** directory named **Startup.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    <span data-ttu-id="142c2-166">Este código permitirá la [inserción de dependencias](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) en las funciones de Azure, exponiendo el `IConfiguration` objeto y el `GraphClientService` servicio.</span><span class="sxs-lookup"><span data-stu-id="142c2-166">This code will enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `IConfiguration` object and the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="142c2-167">Implementar la función GetMyNewestMessage</span><span class="sxs-lookup"><span data-stu-id="142c2-167">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="142c2-168">Abra **./GraphTutorial/GetMyNewestMessage.CS** y reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="142c2-168">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="142c2-169">Revise el código en GetMyNewestMessage.cs</span><span class="sxs-lookup"><span data-stu-id="142c2-169">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="142c2-170">Tómese un momento para considerar lo que hace el código en **GetMyNewestMessage.CS** .</span><span class="sxs-lookup"><span data-stu-id="142c2-170">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="142c2-171">En el constructor, guarda el `IConfiguration` objeto que se pasa a través de la inserción de dependencias.</span><span class="sxs-lookup"><span data-stu-id="142c2-171">In the constructor, it saves the `IConfiguration` object passed in via dependency injection.</span></span>
- <span data-ttu-id="142c2-172">En la `Run` función, hace lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="142c2-172">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="142c2-173">Valida los valores de configuración necesarios que están presentes en el `IConfiguration` objeto.</span><span class="sxs-lookup"><span data-stu-id="142c2-173">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="142c2-174">Valida el token del portador y devuelve un `401` código de estado si el token no es válido.</span><span class="sxs-lookup"><span data-stu-id="142c2-174">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="142c2-175">Obtiene un cliente de Graph del `GraphClientService` para el usuario que realizó esta solicitud.</span><span class="sxs-lookup"><span data-stu-id="142c2-175">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="142c2-176">Usa el SDK de Microsoft Graph para obtener el mensaje más reciente de la bandeja de entrada del usuario y lo devuelve como un cuerpo JSON en la respuesta.</span><span class="sxs-lookup"><span data-stu-id="142c2-176">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="142c2-177">Llamar a la función de Azure desde la aplicación de prueba</span><span class="sxs-lookup"><span data-stu-id="142c2-177">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="142c2-178">Abra **auth.js** y agregue la siguiente función para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="142c2-178">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="142c2-179">Considere lo que hace este código.</span><span class="sxs-lookup"><span data-stu-id="142c2-179">Consider what this code does.</span></span>

    - <span data-ttu-id="142c2-180">En primer lugar, intenta obtener un token de acceso en modo silencioso, sin interacción del usuario.</span><span class="sxs-lookup"><span data-stu-id="142c2-180">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="142c2-181">Como el usuario ya debe haber iniciado sesión, MSAL debe tener tokens para el usuario en su caché.</span><span class="sxs-lookup"><span data-stu-id="142c2-181">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="142c2-182">Si se produce un error que indica que el usuario tiene que interactuar, intenta obtener un token de forma interactiva.</span><span class="sxs-lookup"><span data-stu-id="142c2-182">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

1. <span data-ttu-id="142c2-183">Cree un nuevo archivo en el directorio **TestClient** denominado **azurefunctions.js** y agregue el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="142c2-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="142c2-184">Cambie el directorio actual de la CLI al directorio **./GraphTutorial** y ejecute el siguiente comando para iniciar la función de Azure de forma local.</span><span class="sxs-lookup"><span data-stu-id="142c2-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="142c2-185">Si aún no atiende el SPA, abra una segunda ventana de CLI y cambie el directorio actual al directorio **./TestClient** .</span><span class="sxs-lookup"><span data-stu-id="142c2-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="142c2-186">Ejecute el siguiente comando para ejecutar la aplicación de prueba.</span><span class="sxs-lookup"><span data-stu-id="142c2-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="142c2-187">Abra el explorador y vaya a `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="142c2-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="142c2-188">Inicie sesión y seleccione el **último** elemento de navegación de mensajes.</span><span class="sxs-lookup"><span data-stu-id="142c2-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="142c2-189">La aplicación muestra información acerca del mensaje más reciente en la bandeja de entrada del usuario.</span><span class="sxs-lookup"><span data-stu-id="142c2-189">The app displays information about the newest message in the user's inbox.</span></span>
