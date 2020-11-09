---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822711"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c33c7-101">En este ejercicio obtendrá información sobre los cambios necesarios a la función de Azure de ejemplo para preparar la [publicación en una aplicación de Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span><span class="sxs-lookup"><span data-stu-id="c33c7-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="c33c7-102">Actualización del código</span><span class="sxs-lookup"><span data-stu-id="c33c7-102">Update code</span></span>

<span data-ttu-id="c33c7-103">La configuración se lee desde el almacén de secreto de usuario, que solo se aplica a su equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="c33c7-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="c33c7-104">Antes de publicar en Azure, deberá cambiar el lugar en el que almacena la configuración y actualizar el código en **Startup.CS** en consecuencia.</span><span class="sxs-lookup"><span data-stu-id="c33c7-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Startup.cs** accordingly.</span></span>

<span data-ttu-id="c33c7-105">Los secretos de la aplicación deben almacenarse en almacenamiento seguro, como el [almacén de claves de Azure](https://docs.microsoft.com/azure/key-vault/general/overview).</span><span class="sxs-lookup"><span data-stu-id="c33c7-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="c33c7-106">Actualizar la configuración de CORS para la función de Azure</span><span class="sxs-lookup"><span data-stu-id="c33c7-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="c33c7-107">En este ejemplo, hemos configurado CORS en **local.settings.js** para permitir que la aplicación de prueba llame a la función.</span><span class="sxs-lookup"><span data-stu-id="c33c7-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="c33c7-108">Deberá configurar su función publicada para permitir cualquier aplicación SPA que lo llame.</span><span class="sxs-lookup"><span data-stu-id="c33c7-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="c33c7-109">Actualizar registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="c33c7-109">Update app registrations</span></span>

<span data-ttu-id="c33c7-110">La  `knownClientApplications` propiedad del manifiesto para el registro de la aplicación de la **función de Graph de Azure** tendrá que actualizarse con los identificadores de aplicación de las aplicaciones que llamarán a la función de Azure.</span><span class="sxs-lookup"><span data-stu-id="c33c7-110">The  `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="c33c7-111">Volver a crear las suscripciones existentes</span><span class="sxs-lookup"><span data-stu-id="c33c7-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="c33c7-112">Todas las suscripciones creadas mediante la dirección URL de webhook en el equipo local o ngrok deben volver a crearse con la dirección URL de producción de la `Notify` función de Azure.</span><span class="sxs-lookup"><span data-stu-id="c33c7-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
