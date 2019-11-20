---
title: 資料繫結與 WPF-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: e90d48e6-bea5-47ef-b756-7b89cce4daf0
ms.openlocfilehash: 1933988277d3be8fecc02fced3293f2b7f80c901
ms.sourcegitcommit: ae399f9f3d1bae2c446b552247bd3af3ca5a2cf9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/04/2018
ms.locfileid: "48575661"
---
# <a name="databinding-with-wpf"></a>使用 WPF 資料繫結
此逐步解說示範如何在 [主版詳細資料] 表單中的 WPF 控制項繫結至 POCO 型別。 應用程式會使用 Entity Framework Api 填入資料庫的資料物件、 追蹤變更，並將資料保存到資料庫。

此模型會定義一個對多關聯性中參與的兩種類型：**分類**(主體\\主要) 和**產品**(相依\\詳細資料)。 然後，Visual Studio 工具用來繫結至 WPF 控制項模型中定義的類型。 WPF 資料繫結架構可讓相關的物件之間的瀏覽： 主版檢視中選取的資料列會導致對應的子資料更新的詳細資料檢視。

在本逐步解說的程式碼清單的螢幕擷取畫面取自 Visual Studio 2013，但您可以完成這個逐步解說中的使用 Visual Studio 2012 或 Visual Studio 2010。

## <a name="use-the-object-option-for-creating-wpf-data-sources"></a>使用 'Object' 選項建立的 WPF 資料來源

我們用來建議使用與上一個版本的 Entity Framework**資料庫**選項時建立新的資料來源以使用 EF 設計工具建立的模型為基礎。 這是因為設計工具會產生衍生自 ObjectContext 的內容和實體類別，衍生自 EntityObject。 使用 [資料庫] 選項會幫助您撰寫最佳的程式碼與此 API 介面互動。

Visual Studio 2012 和 Visual Studio 2013 的 EF 設計工具會產生與簡單的 POCO 實體類別是衍生自 DbContext 的內容。 使用 Visual Studio 2010 中，我們建議使用 DbContext，稍後在本逐步解說中所述的程式碼產生範本來交換。

DbContext API 介面時您應該使用**物件**選項建立新的資料來源時，在此逐步解說中所示。

如有需要您可以[還原為基礎的 ObjectContext 的程式碼產生](~/ef6/modeling/designer/codegen/legacy-objectcontext.md)使用 EF 設計工具建立的模型。

## <a name="pre-requisites"></a>必要條件

您必須有 Visual Studio 2013，才能完成此逐步解說安裝的 Visual Studio 2012 或 Visual Studio 2010。

如果您使用 Visual Studio 2010，您也必須安裝 NuGet。 如需詳細資訊，請參閱 <<c0> [ 安裝 NuGet](https://docs.microsoft.com/nuget/install-nuget-client-tools)。  

## <a name="create-the-application"></a>建立應用程式

-   開啟 Visual Studio
-   **檔案-&gt;新增-&gt;專案...**
-   選取  **Windows**的左窗格中並**WPFApplication**在右窗格
-   請輸入**WPFwithEFSample**做為名稱
-   選取 [確定]

## <a name="install-the-entity-framework-nuget-package"></a>安裝 Entity Framework NuGet 套件

-   在 [方案總管] 中，以滑鼠右鍵按一下**WinFormswithEFSample**專案
-   選取**管理 NuGet 封裝...**
-   在 [管理 NuGet 套件] 對話方塊中，選取**線上**索引標籤，然後選擇**EntityFramework**封裝
-   按一下 **安裝**  
    >[!NOTE]
    > 除了 EntityFramework 組件也會加入 System.ComponentModel.DataAnnotations 的參考。 如果專案具有 System.Data.Entity 的參考，則它將會移除安裝 EntityFramework 套件時。 System.Data.Entity 組件不會再用於 Entity Framework 6 應用程式。

## <a name="define-a-model"></a>定義模型

在本逐步解說，您可以選擇實作模型，使用 **Code First(程式碼優先)** 或 **EF 設計工具**。 完成其中兩個下列各節。

### <a name="option-1-define-a-model-using-code-first"></a>選項 1： 定義使用 Code First 模型

本節說明如何建立模型和其相關聯的資料庫使用 Code First。 請跳至下一節 (**選項 2： 定義模型使用 Database First)** 如果您想要使用 Database First 反轉設計您的模型使用 EF 設計工具，從資料庫

使用 Code First 開發時您通常會開始撰寫定義概念 （網域） 模型的.NET Framework 類別。

-   將新類別加入**WPFwithEFSample:**
    -   以滑鼠右鍵按一下專案名稱
    -   選取 **新增**，然後**新項目**
    -   選取 **類別**，然後輸入**Product**作為類別名稱
-   取代**Product**類別定義為下列程式碼：

``` csharp
    namespace WPFwithEFSample
    {
        public class Product
        {
            public int ProductId { get; set; }
            public string Name { get; set; }

            public int CategoryId { get; set; }
            public virtual Category Category { get; set; }
        }
    }
```
-  　新增 **Category** 類別，定義為下列程式碼：
```CSharp
    using System.Collections.ObjectModel;

    namespace WPFwithEFSample
    {
        public class Category
        {
            public Category()
            {
                this.Products = new ObservableCollection<Product>();
            }

            public int CategoryId { get; set; }
            public string Name { get; set; }

            public virtual ObservableCollection<Product> Products { get; private set; }
        }
    }
```

**Category**類別裡的**Product**屬性與**Product**類別裡的**Category**屬性為導覽屬性。 在 Entity Framework 中，導覽屬性會提供瀏覽兩個實體類型之間的關聯性的方法。

除了定義實體，您需要定義類別，衍生自 DbContext，並公開 DbSet&lt;TEntity&gt;屬性。 DbSet&lt;TEntity&gt;屬性可讓知道您想要在模型中包含哪些的類型的內容。

DbContext 衍生型別的執行個體在執行階段，其中包含填入資料庫的資料的物件期間同時管理實體物件、 變更追蹤，和保存資料至資料庫。

-   加入新**ProductContext**類別到專案中的下列定義：

``` csharp
    using System.Data.Entity;

    namespace WPFwithEFSample
    {
        public class ProductContext : DbContext
        {
            public DbSet<Category> Categories { get; set; }
            public DbSet<Product> Products { get; set; }
        }
    }
```

編譯專案。

### <a name="option-2-define-a-model-using-database-first"></a>選項 2： 定義使用 Database First 模型

本節說明如何使用 Database First，進行反向工程，您使用 EF 設計工具，從資料庫的模型。 如果您已完成上一節 (**選項 1： 定義模型使用 Code First)**，然後略過本節並直接前往**消極式載入**一節。

#### <a name="create-an-existing-database"></a>建立現有的資料庫

通常當您的目標將已建立，現有的資料庫，但在此逐步解說，我們需要建立可存取的資料庫。

Visual Studio 隨附安裝的資料庫伺服器是您已安裝的 Visual Studio 版本而有所不同：

-   如果您使用 Visual Studio 2010 將會建立 SQL Express 資料庫。
-   如果您使用 Visual Studio 2012，則您將會建立[LocalDB](https://msdn.microsoft.com/library/hh510202.aspx)資料庫。

接下來產生的資料庫。

-   **檢視-&gt;伺服器總管**
-   以滑鼠右鍵按一下**資料連線-&gt;新增連接...**
-   如果您尚未連線至資料庫從伺服器總管之前您必須選取 Microsoft SQL Server 做為資料來源

    ![變更資料來源](~/ef6/media/changedatasource.png)

-   連接到 LocalDB 或 SQL Express，何者而定，您已安裝，然後輸入**產品**做為資料庫名稱

    ![新增連線 LocalDB](~/ef6/media/addconnectionlocaldb.png)

    ![新增連線 Express](~/ef6/media/addconnectionexpress.png)

-   選取  **確定**而且會要求您想要建立新資料庫，請選取**是**

    ![建立資料庫](~/ef6/media/createdatabase.png)

-   新資料庫現在會出現在 [伺服器總管] 中，按一下滑鼠右鍵，然後選取**新查詢**
-   將下列 SQL 複製到新的查詢，然後以滑鼠右鍵按一下查詢並選取**Execute**

``` SQL
    CREATE TABLE [dbo].[Categories] (
        [CategoryId] [int] NOT NULL IDENTITY,
        [Name] [nvarchar](max),
        CONSTRAINT [PK_dbo.Categories] PRIMARY KEY ([CategoryId])
    )

    CREATE TABLE [dbo].[Products] (
        [ProductId] [int] NOT NULL IDENTITY,
        [Name] [nvarchar](max),
        [CategoryId] [int] NOT NULL,
        CONSTRAINT [PK_dbo.Products] PRIMARY KEY ([ProductId])
    )

    CREATE INDEX [IX_CategoryId] ON [dbo].[Products]([CategoryId])

    ALTER TABLE [dbo].[Products] ADD CONSTRAINT [FK_dbo.Products_dbo.Categories_CategoryId] FOREIGN KEY ([CategoryId]) REFERENCES [dbo].[Categories] ([CategoryId]) ON DELETE CASCADE
```

#### <a name="reverse-engineer-model"></a>反向工程模型

我們將利用 Entity Framework Designer 中，也就是 Visual Studio 的一部分，以建立我們的模型。

-   **專案-&gt;加入新項目...**
-   選取 **資料**從左側功能表，然後**ADO.NET 實體資料模型**
-   請輸入**ProductModel**作為名稱，然後按一下 **[確定]**
-   這會啟動**Entity Data Model 精靈**
-   選取 **從資料庫產生**，按一下 **下一步**

    ![選擇模型內容](~/ef6/media/choosemodelcontents.png)

-   選取您建立第一個區段中的資料庫連接中，輸入**ProductContext**做為連接字串，然後按一下 [名稱**下一步]**

    ![選擇您的連線](~/ef6/media/chooseyourconnection.png)

-   按一下 匯入的所有資料表，並按一下 完成 '資料表旁的核取方塊

    ![選擇您的物件](~/ef6/media/chooseyourobjects.png)

反向工程程序完成後新模型加入您的專案，並讓您檢視在 Entity Framework Designer 中開啟。 App.config 檔案也已新增至您的專案與資料庫的連線詳細資料。

#### <a name="additional-steps-in-visual-studio-2010"></a>Visual Studio 2010 中的其他步驟

如果您正在 Visual Studio 2010 中將需要更新使用 EF6 程式碼產生的 EF 設計工具。

-   以滑鼠右鍵按一下您的模型，在 EF 設計工具中的非空點，然後選取**加入的程式碼產生項目...**
-   選取 **線上範本**從左側的功能表，並搜尋**DbContext**
-   選取**EF 6.x 適用於 C 的 DbContext Generator\#，** 輸入**ProductsModel**做為名稱並按一下 [新增]

#### <a name="updating-code-generation-for-data-binding"></a>更新資料繫結的程式碼產生

EF 從模型使用 T4 範本產生程式碼。 隨附於 Visual Studio，或從 Visual Studio 組件庫下載的範本供一般用途使用。 這表示從這些範本所產生的實體有簡單的 ICollection&lt;T&gt;屬性。 不過，進行資料繫結使用 WPF 時，適合使用**ObservableCollection**集合屬性，因此該 WPF 可以追蹤的集合所做的變更。 為了這個目的，我們會修改範本，以使用 ObservableCollection。

-   開啟**方案總管**並尋找**ProductModel.edmx**檔案
-   尋找**ProductModel.tt**下 ProductModel.edmx 檔案會進行巢狀檔案

    ![WPF 產品模型範本](~/ef6/media/wpfproductmodeltemplate.png)

-   按兩下 ProductModel.tt 檔案在 Visual Studio 編輯器中開啟它
-   尋找和取代出現兩次的 「**ICollection**"with"**ObservableCollection**"。 這些是大約位於線條 296 及 484。
-   尋找和取代第一個出現的 「**HashSet**"with"**ObservableCollection**"。 這項問題位於大約第 50 行。 **不這麼做**取代 HashSet 稍後在程式碼中找到的第二個執行個體。
-   尋找和取代的唯一相符項目 」**System.Collections.Generic**"with"**System.Collections.ObjectModel**"。 這是大約位於行 424。
-   儲存 ProductModel.tt 檔案。 這應該會造成重新產生實體的程式碼。 如果程式碼不會自動重新產生，然後以滑鼠右鍵按一下 ProductModel.tt，並選擇 [執行自訂工具]。

如果您現在開啟 Category.cs 檔案 （這在 ProductModel.tt 之下巢狀），則您應該會看到產品集合中具有類型**ObservableCollection&lt;產品&gt;**。

編譯專案。

## <a name="lazy-loading"></a>消極式載入

**產品**屬性上的**類別目錄**類別並**類別**屬性**產品**類別為導覽屬性。 在 Entity Framework 中，導覽屬性會提供瀏覽兩個實體類型之間的關聯性的方法。

EF 可讓您選擇載入相關的實體從資料庫自動第一次存取導覽屬性。 使用此類型的載入 （稱為消極式載入），請注意，第一次存取每個導覽屬性不同的查詢將會針對資料庫執行如果內容已不在內容中。

使用 POCO 實體類型時，EF 會建立衍生的 proxy 型別的執行個體在執行階段，然後再覆寫虛擬屬性類別，藉此加入載入攔截程序中的達到消極式載入。 若要取得相關物件的消極式載入，您必須宣告瀏覽做為屬性 getter**公用**並**虛擬**(**Overridable** Visual Basic 中)，且您的類別不可以是**密封**(**NotOverridable** Visual Basic 中)。 使用資料庫時的第一個導覽屬性會自動進行虛擬化來啟用消極式載入。 第一個程式碼區段中，我們選擇基於相同原因，請瀏覽屬性虛擬

## <a name="bind-object-to-controls"></a>物件繫結至控制項

加入做為資料來源，此 WPF 應用程式模型中所定義的類別。

-   按兩下**MainWindow.xaml**在 [方案總管] 來開啟主表單
-   從主功能表中，選取**專案-&gt;加入新的資料來源...**
    (在 Visual Studio 2010 中，您必須選取**的資料-&gt;加入新資料來源...**)
-   在 選擇資料來源 Typewindow 中，選取**物件**，按一下 **下一步**
-   在 [選取資料物件] 對話方塊中，展開**WPFwithEFSample**兩個時間和選取**類別**  
    *若要選取不需要**產品**資料來源，因為我們會透過**產品**的屬性**分類**資料來源*  

    ![選取資料物件](~/ef6/media/selectdataobjects.png)

-   按一下 **完成。**
-   [資料來源] 視窗會開啟 [MainWindow.xaml] 視窗旁邊*如果未顯示 [資料來源] 視窗，選取**檢視-&gt;其他 Windows-&gt;資料來源***
-   按 釘選 圖示，讓資料來源 視窗不會不會自動隱藏。 您可能需要按 [重新整理] 按鈕，如果視窗已顯示。

    ![Data Sources](~/ef6/media/datasources.png)

-   選取 **分類**資料來源，並將它拖曳到表單上。

下列狀況時拖曳的項目此來源：

-   **CategoryViewSource**資源並**categoryDataGrid**控制項已加入至 XAML 
-   在父格線項目上的 DataContext 屬性設定為"{StaticResource **categoryViewSource** }"。 **CategoryViewSource**資源做為外部的繫結來源\\父格線項目。 內部的格線項目然後繼承的 DataContext 值從父代 （categoryDataGrid 的 ItemsSource 屬性設定為"{Binding}"） 的方格

``` xml
    <Window.Resources>
        <CollectionViewSource x:Key="categoryViewSource"
                                d:DesignSource="{d:DesignInstance {x:Type local:Category}, CreateList=True}"/>
    </Window.Resources>
    <Grid DataContext="{StaticResource categoryViewSource}">
        <DataGrid x:Name="categoryDataGrid" AutoGenerateColumns="False" EnableRowVirtualization="True"
                    ItemsSource="{Binding}" Margin="13,13,43,191"
                    RowDetailsVisibilityMode="VisibleWhenSelected">
            <DataGrid.Columns>
                <DataGridTextColumn x:Name="categoryIdColumn" Binding="{Binding CategoryId}"
                                    Header="Category Id" Width="SizeToHeader"/>
                <DataGridTextColumn x:Name="nameColumn" Binding="{Binding Name}"
                                    Header="Name" Width="SizeToHeader"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
```

## <a name="adding-a-details-grid"></a>加入詳細資料方格

既然我們已經讓我們來顯示類別目錄方格加入顯示相關聯的產品詳細資料方格。

-   選取 **產品**屬性從底下**分類**資料來源，並將它拖曳到表單上。
    -   **CategoryProductsViewSource**資源並**productDataGrid**方格會新增至 XAML
    -   此資源的繫結路徑設定為產品
    -   WPF 資料繫結架構可確保與所選分類的產品只有顯示在**productDataGrid**
-   從 工具箱 拖曳** 按鈕**入表單。 設定**名稱**屬性設**buttonSave**並**內容**屬性設**儲存**。

表單看起來應該如下所示：

![Designer](~/ef6/media/designer.png) 

## <a name="add-code-that-handles-data-interaction"></a>將會處理與資料互動的程式碼

就可以開始將一些事件處理常式新增至主視窗。

-   在 XAML 視窗中，按一下**&lt;視窗**項目，這會選取的主視窗
-   在 **屬性**視窗中選擇**事件**右上方，然後按兩下右側的文字方塊**Loaded**標籤

    ![主視窗屬性](~/ef6/media/mainwindowproperties.png)

-   也加入**按一下 **事件**儲存**按兩下設計工具中的 儲存 按鈕的按鈕。 

這將帶您前往程式碼後置表單，我們現在要編輯的程式碼，用以 ProductContext 執行資料存取。 更新 MainWindow 的程式碼，如下所示。

程式碼會宣告的長時間執行執行個體**ProductContext**。 **ProductContext**物件用來查詢，並將資料儲存至資料庫。 **Dispose （)** 上**ProductContext**執行個體然後，呼叫來自覆寫**OnClosing**方法。 程式碼註解提供程式碼所執行的作業的相關詳細資料。

``` csharp
    using System.Data.Entity;
    using System.Linq;
    using System.Windows;

    namespace WPFwithEFSample
    {
        public partial class MainWindow : Window
        {
            private ProductContext _context = new ProductContext();
            public MainWindow()
            {
                InitializeComponent();
            }

            private void Window_Loaded(object sender, RoutedEventArgs e)
            {
                System.Windows.Data.CollectionViewSource categoryViewSource =
                    ((System.Windows.Data.CollectionViewSource)(this.FindResource("categoryViewSource")));

                // Load is an extension method on IQueryable,
                // defined in the System.Data.Entity namespace.
                // This method enumerates the results of the query,
                // similar to ToList but without creating a list.
                // When used with Linq to Entities this method
                // creates entity objects and adds them to the context.
                _context.Categories.Load();

                // After the data is loaded call the DbSet<T>.Local property
                // to use the DbSet<T> as a binding source.
                categoryViewSource.Source = _context.Categories.Local;
            }

            private void buttonSave_Click(object sender, RoutedEventArgs e)
            {
                // When you delete an object from the related entities collection
                // (in this case Products), the Entity Framework doesn’t mark
                // these child entities as deleted.
                // Instead, it removes the relationship between the parent and the child
                // by setting the parent reference to null.
                // So we manually have to delete the products
                // that have a Category reference set to null.

                // The following code uses LINQ to Objects
                // against the Local collection of Products.
                // The ToList call is required because otherwise the collection will be modified
                // by the Remove call while it is being enumerated.
                // In most other situations you can use LINQ to Objects directly
                // against the Local property without using ToList first.
                foreach (var product in _context.Products.Local.ToList())
                {
                    if (product.Category == null)
                    {
                        _context.Products.Remove(product);
                    }
                }

                _context.SaveChanges();
                // Refresh the grids so the database generated values show up.
                this.categoryDataGrid.Items.Refresh();
                this.productsDataGrid.Items.Refresh();
            }

            protected override void OnClosing(System.ComponentModel.CancelEventArgs e)
            {
                base.OnClosing(e);
                this._context.Dispose();
            }
        }

    }
```

## <a name="test-the-wpf-application"></a>測試 WPF 應用程式

-   編譯並執行應用程式。 如果您使用 Code First，則您會看到**WPFwithEFSample.ProductContext**為您建立資料庫。
-   在下方方格中的最上層方格和產品名稱中輸入類別名稱*輸入不是任何項目識別碼資料行，因為資料庫會產生主要金鑰*

    ![與新的分類和產品的主視窗](~/ef6/media/screen1.png)

-   按下**儲存** 按鈕，將資料儲存至資料庫

DbContext 的呼叫後方**savechanges （)**，識別碼會填入資料庫產生值。 因為我們會呼叫**Refresh()** 之後**savechanges （)** **DataGrid**控制項會更新含有新值。

![具有識別碼填入的主視窗](~/ef6/media/screen2.png)

## <a name="additional-resources"></a>其他資源

若要深入了解集合使用 WPF 資料繫結，請參閱[本主題](https://docs.microsoft.com/dotnet/framework/wpf/data/data-binding-overview#binding-to-collections)WPF 文件中。  
