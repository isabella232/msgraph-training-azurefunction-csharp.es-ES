---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822729"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3ae5c-101">En este ejercicio, creará tres nuevas aplicaciones de Azure AD con el centro de administración de Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="3ae5c-101">In this exercise you will create three new Azure AD applications using the Azure Active Directory admin center:</span></span>

- <span data-ttu-id="3ae5c-102">Un registro de aplicaciones para la aplicación de una sola página para que pueda iniciar sesión en los usuarios y obtener tokens, lo que permite que la aplicación llame a la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-102">An app registration for the single-page application so that it can sign in users and get tokens allowing the application to call the Azure Function.</span></span>
- <span data-ttu-id="3ae5c-103">Un registro de aplicaciones para la función de Azure que permite usar el [flujo en nombre de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) para intercambiar el token enviado por el Spa para un token que le permita llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-103">An app registration for the Azure Function that allows it to use the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to exchange the token sent by the SPA for a token that will allow it to call Microsoft Graph.</span></span>
- <span data-ttu-id="3ae5c-104">Un registro de aplicaciones para el webhook de Azure function que permite usar el [flujo de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) para llamar a Microsoft Graph sin un usuario.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-104">An app registration for the Azure Function webhook that allows it to use the [client credential flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) to call Microsoft Graph without a user.</span></span>

> [!NOTE]
> <span data-ttu-id="3ae5c-105">Este ejemplo requiere tres registros de aplicaciones porque está implementando el flujo en nombre de y el flujo de credenciales de cliente.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-105">This example requires three app registrations because it is implementing both the on-behalf-of flow and the client credential flow.</span></span> <span data-ttu-id="3ae5c-106">Si la función de Azure solo usa uno de estos flujos, solo tendrá que crear los registros de la aplicación que correspondan a ese flujo.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-106">If your Azure Function only uses one of these flows, you would only need to create the app registrations that correspond to that flow.</span></span>

1. <span data-ttu-id="3ae5c-107">Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com) e inicie sesión con un administrador de la organización del arrendatario de Microsoft 365.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using an Microsoft 365 tenant organization admin.</span></span>

1. <span data-ttu-id="3ae5c-108">Seleccione **Azure Active Directory** en el panel de navegación izquierdo y, a continuación, seleccione **Registros de aplicaciones** en **Administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-108">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="3ae5c-109">Una captura de pantalla de los registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="3ae5c-109">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a><span data-ttu-id="3ae5c-110">Registrar una aplicación para la aplicación de una sola página</span><span class="sxs-lookup"><span data-stu-id="3ae5c-110">Register an app for the single-page application</span></span>

1. <span data-ttu-id="3ae5c-111">Seleccione **Nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-111">Select **New registration**.</span></span> <span data-ttu-id="3ae5c-112">En la página **Registrar una aplicación** , establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="3ae5c-113">Establezca **Nombre** como `Graph Azure Function Test App`.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-113">Set **Name** to `Graph Azure Function Test App`.</span></span>
    - <span data-ttu-id="3ae5c-114">Establezca los **tipos de cuenta admitidos** **solo en las cuentas de este directorio de organización**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-114">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="3ae5c-115">En **URI de redireccionamiento** , cambie la opción desplegable a **aplicación de una sola página (Spa)** y establezca el valor en `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="3ae5c-115">Under **Redirect URI** , change the dropdown to **Single-page application (SPA)** and set the value to `http://localhost:8080`.</span></span>

    ![Captura de pantalla de la página registrar una aplicación](./images/register-command-line-app.png)

1. <span data-ttu-id="3ae5c-117">Seleccione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-117">Select **Register**.</span></span> <span data-ttu-id="3ae5c-118">En la página aplicación de prueba de la **función de Azure Graph** , copie los valores del identificador de la **aplicación (cliente)** y el **identificador del directorio (inquilino)** y guárdelos, los necesitará en los pasos posteriores.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-118">On the **Graph Azure Function Test App** page, copy the values of the **Application (client) ID** and **Directory (tenant) ID** and save them, you will need them in the later steps.</span></span>

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a><span data-ttu-id="3ae5c-120">Registrar una aplicación para la función de Azure</span><span class="sxs-lookup"><span data-stu-id="3ae5c-120">Register an app for the Azure Function</span></span>

1. <span data-ttu-id="3ae5c-121">Vuelva a **registros de aplicaciones** y seleccione **nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-121">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="3ae5c-122">En la página **Registrar una aplicación** , establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-122">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="3ae5c-123">Establezca **Nombre** como `Graph Azure Function`.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-123">Set **Name** to `Graph Azure Function`.</span></span>
    - <span data-ttu-id="3ae5c-124">Establezca los **tipos de cuenta admitidos** **solo en las cuentas de este directorio de organización**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-124">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="3ae5c-125">Deje el **URI de redireccionamiento** en blanco.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-125">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="3ae5c-126">Seleccione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-126">Select **Register**.</span></span> <span data-ttu-id="3ae5c-127">En la página de la **función Graph Azure** , copie el valor del identificador de la **aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-127">On the **Graph Azure Function** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="3ae5c-128">Seleccione **Certificados y secretos** en **Administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="3ae5c-129">Seleccione el botón **Nuevo secreto de cliente**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-129">Select the **New client secret** button.</span></span> <span data-ttu-id="3ae5c-130">Escriba un valor en **Descripción** y seleccione una de las opciones para **Expires** y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-130">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    ![Captura de pantalla del cuadro de diálogo Agregar un secreto de cliente](./images/aad-new-client-secret.png)

1. <span data-ttu-id="3ae5c-132">Copie el valor del secreto de cliente antes de salir de esta página.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="3ae5c-133">Lo necesitará en el siguiente paso.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="3ae5c-134">El secreto de cliente no se vuelve a mostrar, así que asegúrese de copiarlo en este momento.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Captura de pantalla del secreto de cliente recién agregado](./images/aad-copy-client-secret.png)

1. <span data-ttu-id="3ae5c-136">Seleccione **permisos de API** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-136">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="3ae5c-137">Elija **Agregar un permiso**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-137">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="3ae5c-138">Seleccione **Microsoft Graph** y, a continuación, **permisos delegados**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-138">Select **Microsoft Graph** , then **Delegated Permissions**.</span></span> <span data-ttu-id="3ae5c-139">Agregue **mail. Read** y seleccione **Add Permissions**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-139">Add **Mail.Read** and select **Add permissions**.</span></span>

    ![Una captura de pantalla de los permisos configurados para el registro de aplicaciones de la función de Azure](./images/web-api-configured-permissions.png)

1. <span data-ttu-id="3ae5c-141">Seleccione **exponer una API** en **administrar** y, a continuación, elija **Agregar un ámbito**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-141">Select **Expose an API** under **Manage** , then choose **Add a scope**.</span></span>

1. <span data-ttu-id="3ae5c-142">Acepte el **URI de identificador de aplicación** predeterminado y elija **Guardar y continuar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-142">Accept the default **Application ID URI** and choose **Save and continue**.</span></span>

1. <span data-ttu-id="3ae5c-143">Rellene el formulario **Agregar un ámbito** de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="3ae5c-143">Fill in the **Add a scope** form as follows:</span></span>

    - <span data-ttu-id="3ae5c-144">**Nombre de ámbito:** Mail. Read</span><span class="sxs-lookup"><span data-stu-id="3ae5c-144">**Scope name:** Mail.Read</span></span>
    - <span data-ttu-id="3ae5c-145">¿ **Quién puede dar su consentimiento?:** Administradores y usuarios</span><span class="sxs-lookup"><span data-stu-id="3ae5c-145">**Who can consent?:** Admins and users</span></span>
    - <span data-ttu-id="3ae5c-146">**Nombre para mostrar de consentimiento de administración:** Leer las bandejas de correo de todos los usuarios</span><span class="sxs-lookup"><span data-stu-id="3ae5c-146">**Admin consent display name:** Read all users' inboxes</span></span>
    - <span data-ttu-id="3ae5c-147">**Descripción del consentimiento del administrador:** Permite que la aplicación Lea las bandejas de correo de todos los usuarios.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-147">**Admin consent description:** Allows the app to read all users' inboxes</span></span>
    - <span data-ttu-id="3ae5c-148">**Nombre para mostrar del consentimiento del usuario:** Leer la bandeja de entrada</span><span class="sxs-lookup"><span data-stu-id="3ae5c-148">**User consent display name:** Read your inbox</span></span>
    - <span data-ttu-id="3ae5c-149">**Descripción del consentimiento del usuario:** Permite que la aplicación lea su bandeja de entrada</span><span class="sxs-lookup"><span data-stu-id="3ae5c-149">**User consent description:** Allows the app to read your inbox</span></span>
    - <span data-ttu-id="3ae5c-150">**Estado:** Preparado</span><span class="sxs-lookup"><span data-stu-id="3ae5c-150">**State:** Enabled</span></span>

1. <span data-ttu-id="3ae5c-151">Seleccione **Agregar ámbito**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-151">Select **Add scope**.</span></span>

1. <span data-ttu-id="3ae5c-152">Copie el nuevo ámbito, lo necesitará en pasos posteriores.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-152">Copy the new scope, you'll need it in later steps.</span></span>

    ![Una captura de pantalla de los ámbitos definidos para el registro de aplicaciones de la función de Azure](./images/web-api-defined-scopes.png)

1. <span data-ttu-id="3ae5c-154">Seleccione **manifiesto** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-154">Select **Manifest** under **Manage**.</span></span>

1. <span data-ttu-id="3ae5c-155">Busque `knownClientApplications` en el manifiesto y reemplace su valor actual `[]` por `[TEST_APP_ID]` , donde `TEST_APP_ID` es el identificador de aplicación del registro de aplicaciones de la aplicación de prueba de la **función de Azure de Graph** .</span><span class="sxs-lookup"><span data-stu-id="3ae5c-155">Locate `knownClientApplications` in the manifest, and replace it's current value of `[]` with `[TEST_APP_ID]`, where `TEST_APP_ID` is the application ID of the **Graph Azure Function Test App** app registration.</span></span> <span data-ttu-id="3ae5c-156">Seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-156">Select **Save**.</span></span>

> [!NOTE]
> <span data-ttu-id="3ae5c-157">Agregar el identificador de aplicación de la aplicación de prueba a la `knownClientApplications` propiedad en el manifiesto de la función de Azure permite que la aplicación de prueba desencadene un [flujo de consentimiento combinado](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span><span class="sxs-lookup"><span data-stu-id="3ae5c-157">Adding the test application's app ID to the `knownClientApplications` property in the Azure Function's manifest allows the test application to trigger a [combined consent flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span></span> <span data-ttu-id="3ae5c-158">Esto es necesario para que funcione el flujo en nombre de.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-158">This is necessary for the on-behalf-of flow to work.</span></span>

## <a name="add-azure-function-scope-to-test-application-registration"></a><span data-ttu-id="3ae5c-159">Agregar ámbito de función de Azure para probar el registro de aplicaciones</span><span class="sxs-lookup"><span data-stu-id="3ae5c-159">Add Azure Function scope to test application registration</span></span>

1. <span data-ttu-id="3ae5c-160">Vuelva al registro de la **aplicación de prueba** de la función de Azure y seleccione permisos de **API** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-160">Return to the **Graph Azure Function Test App** registration, and select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="3ae5c-161">Seleccione **Agregar un permiso**</span><span class="sxs-lookup"><span data-stu-id="3ae5c-161">Select **Add a permission**.</span></span>

1. <span data-ttu-id="3ae5c-162">Seleccione **mis API** y, después, seleccione **cargar más**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-162">Select **My APIs** , then select **Load more**.</span></span> <span data-ttu-id="3ae5c-163">Seleccione la **función de Microsoft Graph Azure**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-163">Select **Graph Azure Function**.</span></span>

    ![Captura de pantalla del cuadro de diálogo solicitar permisos de la API](./images/test-app-add-permissions.png)

1. <span data-ttu-id="3ae5c-165">Seleccione el permiso **mail. Read** y, a continuación, seleccione **Agregar permisos**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-165">Select the **Mail.Read** permission, then select **Add permissions**.</span></span>

1. <span data-ttu-id="3ae5c-166">En los **permisos configurados** , quite el permiso **User. Read** en **Microsoft Graph** seleccionando **...** a la derecha del permiso y seleccionando **quitar permiso**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-166">In the **Configured permissions** , remove the **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="3ae5c-167">Seleccione **sí, quitar** para confirmar.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-167">Select **Yes, remove** to confirm.</span></span>

    ![Una captura de pantalla de los permisos configurados para el registro de la aplicación probar aplicación](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a><span data-ttu-id="3ae5c-169">Registrar una aplicación para el webhook de la función de Azure</span><span class="sxs-lookup"><span data-stu-id="3ae5c-169">Register an app for the Azure Function webhook</span></span>

1. <span data-ttu-id="3ae5c-170">Vuelva a **registros de aplicaciones** y seleccione **nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-170">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="3ae5c-171">En la página **Registrar una aplicación** , establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-171">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="3ae5c-172">Establezca **Nombre** como `Graph Azure Function Webhook`.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-172">Set **Name** to `Graph Azure Function Webhook`.</span></span>
    - <span data-ttu-id="3ae5c-173">Establezca los **tipos de cuenta admitidos** **solo en las cuentas de este directorio de organización**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-173">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="3ae5c-174">Deje el **URI de redireccionamiento** en blanco.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-174">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="3ae5c-175">Seleccione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-175">Select **Register**.</span></span> <span data-ttu-id="3ae5c-176">En la página **webhook de la función de Azure Graph** , copie el valor del identificador de la **aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-176">On the **Graph Azure Function webhook** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="3ae5c-177">Seleccione **Certificados y secretos** en **Administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-177">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="3ae5c-178">Seleccione el botón **Nuevo secreto de cliente**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-178">Select the **New client secret** button.</span></span> <span data-ttu-id="3ae5c-179">Escriba un valor en **Descripción** y seleccione una de las opciones para **Expires** y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-179">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

1. <span data-ttu-id="3ae5c-180">Copie el valor del secreto de cliente antes de salir de esta página.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-180">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="3ae5c-181">Lo necesitará en el siguiente paso.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-181">You will need it in the next step.</span></span>

1. <span data-ttu-id="3ae5c-182">Seleccione **permisos de API** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-182">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="3ae5c-183">Elija **Agregar un permiso**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-183">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="3ae5c-184">Seleccione **Microsoft Graph** y, a continuación, permisos de la **aplicación**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-184">Select **Microsoft Graph** , then **Application Permissions**.</span></span> <span data-ttu-id="3ae5c-185">Agregue **User. Read. All** y **mail. Read** y, a continuación, seleccione **Agregar permisos**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-185">Add **User.Read.All** and **Mail.Read** , then select **Add permissions**.</span></span>

1. <span data-ttu-id="3ae5c-186">En los **permisos configurados** , quite el **usuario delegado.** permiso de lectura en **Microsoft Graph** seleccionando **...** a la derecha del permiso y seleccionando **quitar permiso**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-186">In the **Configured permissions** , remove the delegated **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="3ae5c-187">Seleccione **sí, quitar** para confirmar.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-187">Select **Yes, remove** to confirm.</span></span>

1. <span data-ttu-id="3ae5c-188">Seleccione el botón **conceder consentimiento del administrador para...** y seleccione **sí** para conceder el consentimiento del administrador para los permisos de la aplicación configurada.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-188">Select the **Grant admin consent for...** button, then select **Yes** to grant admin consent for the configured application permissions.</span></span> <span data-ttu-id="3ae5c-189">La columna **Estado** de la tabla **permisos configurados** cambia a **concedido para...**.</span><span class="sxs-lookup"><span data-stu-id="3ae5c-189">The **Status** column in the **Configured permissions** table changes to **Granted for ...**.</span></span>

    ![Una captura de pantalla de los permisos configurados para el webhook con consentimiento de administrador concedido](./images/webhook-configured-permissions.png)
