---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822688"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="93f81-101">Este tutorial le enseña a crear una función de Azure que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.</span><span class="sxs-lookup"><span data-stu-id="93f81-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="93f81-102">Si prefiere descargar solo el tutorial completo, puede descargar o clonar el repositorio de [GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="93f81-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="93f81-103">Consulte el archivo Léame de la carpeta **Demo** para obtener instrucciones sobre cómo configurar la aplicación con un identificador de aplicación y un secreto.</span><span class="sxs-lookup"><span data-stu-id="93f81-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="93f81-104">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="93f81-104">Prerequisites</span></span>

<span data-ttu-id="93f81-105">Antes de iniciar este tutorial, debe tener las siguientes herramientas instaladas en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="93f81-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="93f81-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="93f81-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="93f81-107">Herramientas básicas de Azure Functions</span><span class="sxs-lookup"><span data-stu-id="93f81-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="93f81-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="93f81-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="93f81-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="93f81-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="93f81-110">También debe tener una cuenta profesional o educativa de Microsoft, con acceso a una cuenta de administrador global de la misma organización.</span><span class="sxs-lookup"><span data-stu-id="93f81-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="93f81-111">Si no tiene una cuenta de Microsoft, puede [suscribirse al programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="93f81-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="93f81-112">Este tutorial se ha redactado con las siguientes versiones de las herramientas anteriores.</span><span class="sxs-lookup"><span data-stu-id="93f81-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="93f81-113">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="93f81-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="93f81-114">.NET Core SDK 3.1.301</span><span class="sxs-lookup"><span data-stu-id="93f81-114">.NET Core SDK 3.1.301</span></span>
> - <span data-ttu-id="93f81-115">Herramientas básicas de Azure Functions 3.0.2630</span><span class="sxs-lookup"><span data-stu-id="93f81-115">Azure Functions Core Tools 3.0.2630</span></span>
> - <span data-ttu-id="93f81-116">Azure CLI 2.8.0</span><span class="sxs-lookup"><span data-stu-id="93f81-116">Azure CLI 2.8.0</span></span>
> - <span data-ttu-id="93f81-117">ngrok 2.3.35</span><span class="sxs-lookup"><span data-stu-id="93f81-117">ngrok 2.3.35</span></span>

## <a name="feedback"></a><span data-ttu-id="93f81-118">Comentarios</span><span class="sxs-lookup"><span data-stu-id="93f81-118">Feedback</span></span>

<span data-ttu-id="93f81-119">Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="93f81-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
