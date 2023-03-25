using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace DataBindingDemoF
{
    static class Program
    {
        /// <summary>
        /// The main entry point for the application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new MainForm());
        }
    }
}
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace DataBindingDemoF
{
    class ProductViewModel : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        public readonly decimal GstRate;
        public readonly decimal PstRate;

        protected void OnPropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        public ProductViewModel(decimal gstRate, decimal pstRate)
        {
            GstRate = gstRate;
            PstRate = pstRate;
            products = DataGenerator.CreateProducts(GstRate, PstRate);
            ProductsSource = new BindingSource();
            ProductsSource.DataSource = products;
            ProductsSource.ListChanged += ProductSource_ListChanged;
            DisplayProduct = new Product(GstRate, PstRate);
        }

        private Product displayProduct;
        public Product DisplayProduct
        {
            get { return displayProduct; }
            set
            {
                //Create a copy of the product as a temporary placeholder
                //so that the original product doesn't get updated until we hit save
                displayProduct = new Product(GstRate, PstRate)
                {
                    ProductId = value.ProductId,
                    Sku = value.Sku,
                    Description = value.Description,
                    Cost = value.Cost,
                    IsTaxable = value.IsTaxable
                };
                OnPropertyChanged();
            }
        }
        private readonly ProductList products;
        public BindingSource ProductsSource { get; }

        //Can't access count directly when databinding due to a limitation in the framework.
        //So, we'll use this property to pass it through.
        public int Count { 
            get { 
                return products.Count; 
            } 
        }

        public string Totals
        {
            get
            {
                return string.Format("{0,15:N2}\r\n{1,15:N2}\r\n{2,15:N2}\r\n{3,15:N2}\r\n{4,15:N2}\r\n",
                                                   products.TotalCost,
                                                   "Placeholder",
                                                   "Placeholder",
                                                   "Placeholder",
                                                   "Placeholder");
            }
        } 

        //If the list has changed, that means something was either added, updated, or deleted.
        //When this happens, we need to raise events to tell the controls that there has been an
        //update so that they know to refresh their values.
        private void ProductSource_ListChanged(object sender, ListChangedEventArgs e)
        {
            OnPropertyChanged("Count");
            OnPropertyChanged("Totals");
        }

    }
}
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace DataBindingDemoF
{
    class ProductList : List<Product>
    {
        public decimal TotalCost
        {
            get
            {
                decimal totalCost = 0.0m;
                foreach(Product product in this)
                {
                    totalCost += product.ExtensionCost;
                }
                return totalCost;
            }
        }
    }
}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace DataBindingDemoF
{
    class Product
    {
        public decimal PstRate { get; }
        public decimal GstRate { get; }
        public int ProductId { get; set; }
        public string Sku { get; set; }
        public string Description { get; set; }
        public decimal Cost { get; set; }
        public bool IsTaxable { get; set; }
        public decimal ExtensionCost
        {
            get
            {
                return Cost * 1;
            }
        }
        public Product(decimal pstRate, decimal gstRate)
        {
            PstRate = pstRate;
            GstRate = gstRate;
        }
    }
}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace DataBindingDemoF
{
    class DataGenerator
    {
        public static ProductList CreateProducts(decimal gstRate, decimal pstRate)
        {
            ProductList products = new ProductList();

            products.Add(new Product(gstRate, pstRate) { ProductId = 1, Sku = "ABC100", Description = "Nice Widget 1", Cost = 452.55m, IsTaxable = true });
            products.Add(new Product(gstRate, pstRate) { ProductId = 2, Sku = "ABC120", Description = "Nice Widget 2", Cost = 652.25m, IsTaxable = true });
            products.Add(new Product(gstRate, pstRate) { ProductId = 3, Sku = "BDC140", Description = "Nice Widget 3", Cost = 1256.00m, IsTaxable = true });
            products.Add(new Product(gstRate, pstRate) { ProductId = 4, Sku = "BDC180", Description = "Nice Widget 4", Cost = 874.25m, IsTaxable = true });
            products.Add(new Product(gstRate, pstRate) { ProductId = 5, Sku = "FAC205", Description = "Nice Widget 5", Cost = 559.22m, IsTaxable = true });
            products.Add(new Product(gstRate, pstRate) { ProductId = 6, Sku = "GBS300", Description = "Nice Widget 6", Cost = 52.05m, IsTaxable = false });

            return products;
        }
    }
}
