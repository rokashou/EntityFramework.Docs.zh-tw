---
title: 使用個別的遷移專案-EF Core
author: bricelam
ms.author: bricelam
ms.date: 10/30/2017
uid: core/managing-schemas/migrations/projects
ms.openlocfilehash: 0c08855db77470d28e23f9ef1d147497dfcdff83
ms.sourcegitcommit: 18ab4c349473d94b15b4ca977df12147db07b77f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/06/2019
ms.locfileid: "73655560"
---
# <a name="using-a-separate-migrations-project"></a>使用個別的遷移專案

您可能想要將您的遷移儲存在與包含您的 `DbContext`不同的元件中。 您也可以使用此策略來維護多組遷移，例如，一個用於開發，另一個用於發行至發行升級。

若要執行相關作業…

1. 建立新的類別庫。

2. 新增 DbCoNtext 元件的參考。

3. 將「遷移」和「模型快照集」檔案移至類別庫。
   > [!TIP]
   > 如果您沒有任何現有的遷移，請在包含 DbCoNtext 的專案中產生一個，然後將它移動。
   > 這很重要，因為如果遷移元件不包含現有的遷移，則新增遷移命令將找不到 DbCoNtext。

4. 設定遷移元件：

   ``` csharp
   options.UseSqlServer(
       connectionString,
       x => x.MigrationsAssembly("MyApp.Migrations"));
   ```

5. 從啟動元件新增對您的遷移元件的參考。
   * 如果這會造成迴圈相依性，請更新類別庫的輸出路徑：

     ``` xml
     <PropertyGroup>
       <OutputPath>..\MyStartupProject\bin\$(Configuration)\</OutputPath>
     </PropertyGroup>
     ```

如果您已正確執行所有工作，您應該能夠將新的遷移新增至專案。

## <a name="net-core-clitabdotnet-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

``` Console
dotnet ef migrations add NewMigration --project MyApp.Migrations
```

## <a name="visual-studiotabvs"></a>[Visual Studio](#tab/vs)

``` powershell
Add-Migration NewMigration -Project MyApp.Migrations
```

***
