---
title: Azure Cosmos DB 提供者 - EF Core
description: 資料庫提供者的文件，內容說明如何搭配使用 Entity Framework Core 與 Azure Cosmos DB SQL API
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 11/05/2019
uid: core/providers/cosmos/index
ms.openlocfilehash: 6cac695288d9ba84968b7fab6361f55e9b51be67
ms.sourcegitcommit: 18ab4c349473d94b15b4ca977df12147db07b77f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/06/2019
ms.locfileid: "73656083"
---
# <a name="ef-core-azure-cosmos-db-provider"></a>EF Core Azure Cosmos DB 提供者

>[!NOTE]
> 這是新的 EF Core 3.0 提供者。

此資料庫提供者可讓 Entity Framework Core 與 Azure Cosmos DB 搭配使用。 [Entity Framework Core 專案](https://github.com/aspnet/EntityFrameworkCore)的維護包含此提供者。

強烈建議您在閱讀本節之前先熟悉 [Azure Cosmos DB 文件](/azure/cosmos-db/introduction)。

>[!NOTE]
> 只能搭配 Azure Cosmos DB 的 SQL API 使用此提供者。

## <a name="install"></a>安裝

安裝 [Microsoft.EntityFrameworkCore.Cosmos NuGet 套件](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos/)。

## <a name="net-core-clitabdotnet-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

``` console
dotnet add package Microsoft.EntityFrameworkCore.Cosmos
```

## <a name="visual-studiotabvs"></a>[Visual Studio](#tab/vs)

``` powershell
Install-Package Microsoft.EntityFrameworkCore.Cosmos
```

***

## <a name="get-started"></a>開始使用

> [!TIP]  
> 您可以檢視本文中的 [GitHut 範例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Cosmos)。

與其他提供者一樣，第一步是呼叫 [UseCosmos](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosDbContextOptionsExtensions.UseCosmos)：

[!code-csharp[Configuration](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=Configuration)]

> [!WARNING]
> 為了簡便，此處將端點和索引鍵進行了硬式編碼，但應在生產應用程式中[以安全的方式儲存它們](/aspnet/core/security/app-secrets#secret-manager)

在此範例中，`Order` 是簡單實體，其參考了[擁有的類型](../../modeling/owned-entities.md) `StreetAddress`。

[!code-csharp[Order](../../../../samples/core/Cosmos/ModelBuilding/Order.cs?name=Order)]

[!code-csharp[StreetAddress](../../../../samples/core/Cosmos/ModelBuilding/StreetAddress.cs?name=StreetAddress)]

遵循一般的 EF 模式儲存及查詢資料：

[!code-csharp[HelloCosmos](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=HelloCosmos)]

> [!IMPORTANT]
> 如果要建立需要的容器或插入[種子資料](../../modeling/data-seeding.md) (如果存在於模型中)，就必須呼叫 [EnsureCreatedAsync](/dotnet/api/Microsoft.EntityFrameworkCore.Storage.IDatabaseCreator.EnsureCreatedAsync)。 但是，`EnsureCreatedAsync` 僅應在部署期間呼叫，在一般作業期間呼叫可能會導致效能問題。

## <a name="cosmos-specific-model-customization"></a>Cosmos 專用模型自訂

根據預設，所有實體類型都會對應至相同容器，並以衍生的內容命名 (在此案例中為 `"OrderContext"`)。 若要變更預設容器名稱，請使用 [HasDefaultContainer](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosModelBuilderExtensions.HasDefaultContainer)：

[!code-csharp[DefaultContainer](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=DefaultContainer)]

若要將實體類型對應至不同的容器，請使用 [ToContainer](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.ToContainer)：

[!code-csharp[Container](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=Container)]

為了識別其指定項目代表 EF Core 的實體類型，即使沒有衍生的實體類型，也會新增鑑別子值。 鑑別子的名稱和值[可以變更](../../modeling/inheritance.md)。

如果不會在同一個容器中儲存其他實體類型，就可以呼叫 [HasNoDiscriminator](/dotnet/api/Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder.HasNoDiscriminator) 將鑑別子移除：

[!code-csharp[NoDiscriminator](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=NoDiscriminator)]

### <a name="partition-keys"></a>資料分割索引鍵

根據預設，EF Core 會使用對 `"__partitionKey"` 設定的分割區索引鍵來建立容器，但不會在插入項目時提供任何值。 不過，如果要充分利用 Azure Cosmos 的效能功能，就必須使用[謹慎選取的分割區索引鍵](/azure/cosmos-db/partition-data)。 您可以透過呼叫 [HasPartitionKey](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.HasPartitionKey) 來設定分割區索引鍵：

[!code-csharp[PartitionKey](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=PartitionKey)]

>[!NOTE]
>只要分割區索引鍵屬性會[轉換成字串](xref:core/modeling/value-conversions)，就不限於任何類型。

分割區索引鍵屬性設定完成後，應該都會具有 null 以外的值。 您可在發出查詢時新增條件，讓查詢成為單一分割區。

[!code-csharp[PartitionKey](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=PartitionKey)]

## <a name="embedded-entities"></a>內嵌實體

針對 Cosmos，擁有的實體會以擁有者身分內嵌在相同項目中。 若要變更屬性名稱，請使用 [ToJsonProperty](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.ToJsonProperty)：

[!code-csharp[PropertyNames](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=PropertyNames)]

使用此組態，來自以上範例的順序會以如下方式儲存：

``` json
{
    "Id": 1,
    "PartitionKey": "1",
    "TrackingNumber": null,
    "id": "1",
    "Address": {
        "ShipsToCity": "London",
        "ShipsToStreet": "221 B Baker St"
    },
    "_rid": "6QEKAM+BOOABAAAAAAAAAA==",
    "_self": "dbs/6QEKAA==/colls/6QEKAM+BOOA=/docs/6QEKAM+BOOABAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-683c-692e763901d5\"",
    "_attachments": "attachments/",
    "_ts": 1568163674
}
```

同時也會內嵌自有實體的集合。 在下一個範例中，我們將使用 `Distributor` 類別和 `StreetAddress` 的集合：

[!code-csharp[Distributor](../../../../samples/core/Cosmos/ModelBuilding/Distributor.cs?name=Distributor)]

自有實體不需提供明確索引鍵值來儲存：

[!code-csharp[OwnedCollection](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=OwnedCollection)]

它們會以如下方式保存：

``` json
{
    "Id": 1,
    "Discriminator": "Distributor",
    "id": "Distributor|1",
    "ShippingCenters": [
        {
            "City": "Phoenix",
            "Street": "500 S 48th Street"
        },
        {
            "City": "Anaheim",
            "Street": "5650 Dolly Ave"
        }
    ],
    "_rid": "6QEKANzISj0BAAAAAAAAAA==",
    "_self": "dbs/6QEKAA==/colls/6QEKANzISj0=/docs/6QEKANzISj0BAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-683c-7b2b439701d5\"",
    "_attachments": "attachments/",
    "_ts": 1568163705
}
```

在內部，EF Core 一律需要具有所有追蹤實體的唯一索引鍵值。 根據預設，為擁有類型的集合所建立主索引鍵，由指向擁有者的外部索引鍵屬性和與 JSON 陣列中索引對應的 `int` 屬性組成。 若要擷取這些值，可以使用項目 API：

[!code-csharp[ImpliedProperties](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=ImpliedProperties)]

> [!TIP]
> 必要時，您可以變更自有實體類型的預設主索引鍵，但應明確提供索引鍵值。

## <a name="working-with-disconnected-entities"></a>使用已中斷連線的實體

每個項目都必須具有 `id` 值，該值對於特定分割區索引鍵是唯一的。 根據預設，EF Core 會使用 “|” 作為分隔符號，將鑑別子和主索引鍵值相結合以產生值。 僅當實體進入 `Added` 狀態時才會產生索引鍵值。 在[連結實體](../../saving/disconnected-entities.md)時，如果它們在 .NET 類型上不具有 `id` 屬性來儲存值，則可能會發生問題。

若要解決此限制，您可以手動建立並設定 `id` 值，或先將實體標記為已新增，然後將其變更為所需狀態：

[!code-csharp[Attach](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?highlight=4&name=Attach)]

以下是輸出 JSON：

``` json
{
    "Id": 1,
    "Discriminator": "Distributor",
    "id": "Distributor|1",
    "ShippingCenters": [
        {
            "City": "Phoenix",
            "Street": "500 S 48th Street"
        }
    ],
    "_rid": "JBwtAN8oNYEBAAAAAAAAAA==",
    "_self": "dbs/JBwtAA==/colls/JBwtAN8oNYE=/docs/JBwtAN8oNYEBAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-9377-d7a1ae7c01d5\"",
    "_attachments": "attachments/",
    "_ts": 1572917100
}
```
