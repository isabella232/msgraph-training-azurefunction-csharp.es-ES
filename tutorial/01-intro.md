---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822688"
---
<!-- markdownlint-disable MD002 MD041 -->

Este tutorial le enseña a crear una función de Azure que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.

> [!TIP]
> Si prefiere descargar solo el tutorial completo, puede descargar o clonar el repositorio de [GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp). Consulte el archivo Léame de la carpeta **Demo** para obtener instrucciones sobre cómo configurar la aplicación con un identificador de aplicación y un secreto.

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener las siguientes herramientas instaladas en el equipo de desarrollo.

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Herramientas básicas de Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

También debe tener una cuenta profesional o educativa de Microsoft, con acceso a una cuenta de administrador global de la misma organización. Si no tiene una cuenta de Microsoft, puede [suscribirse al programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se ha redactado con las siguientes versiones de las herramientas anteriores. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.
>
> - .NET Core SDK 3.1.301
> - Herramientas básicas de Azure Functions 3.0.2630
> - Azure CLI 2.8.0
> - ngrok 2.3.35

## <a name="feedback"></a>Comentarios

Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).
