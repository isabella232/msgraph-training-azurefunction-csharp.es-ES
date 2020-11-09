---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822711"
---
<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio obtendrá información sobre los cambios necesarios a la función de Azure de ejemplo para preparar la [publicación en una aplicación de Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).

## <a name="update-code"></a>Actualización del código

La configuración se lee desde el almacén de secreto de usuario, que solo se aplica a su equipo de desarrollo. Antes de publicar en Azure, deberá cambiar el lugar en el que almacena la configuración y actualizar el código en **Startup.CS** en consecuencia.

Los secretos de la aplicación deben almacenarse en almacenamiento seguro, como el [almacén de claves de Azure](https://docs.microsoft.com/azure/key-vault/general/overview).

## <a name="update-cors-setting-for-azure-function"></a>Actualizar la configuración de CORS para la función de Azure

En este ejemplo, hemos configurado CORS en **local.settings.js** para permitir que la aplicación de prueba llame a la función. Deberá configurar su función publicada para permitir cualquier aplicación SPA que lo llame.

## <a name="update-app-registrations"></a>Actualizar registros de la aplicación

La  `knownClientApplications` propiedad del manifiesto para el registro de la aplicación de la **función de Graph de Azure** tendrá que actualizarse con los identificadores de aplicación de las aplicaciones que llamarán a la función de Azure.

## <a name="recreate-existing-subscriptions"></a>Volver a crear las suscripciones existentes

Todas las suscripciones creadas mediante la dirección URL de webhook en el equipo local o ngrok deben volver a crearse con la dirección URL de producción de la `Notify` función de Azure.
