// App state
let categories = [];
let products = {};  // Structured as {categoryId: [products]}
let currentCategory = null;
let currentEditingProduct = null;
let dollarRate = 13000; // Default value
let dataChanged = false; // Track if data has been changed
let searchTerm = ''; // Current search term

// DOM elements - Main UI
const dollarRateInput = document.getElementById('dollar-rate');
const homeBtn = document.getElementById('home-btn');
const categoryBreadcrumb = document.getElementById('category-breadcrumb');
const categoryNameDisplay = document.getElementById('category-name-display');
const editCategoryNameBtn = document.getElementById('edit-category-name-btn');
const saveDataBtn = document.getElementById('save-data-btn');
const searchInput = document.getElementById('search-input');
const clearSearchBtn = document.getElementById('clear-search');
const searchResultsCount = document.getElementById('search-results-count');
const exportDataBtn = document.getElementById('export-data');
const importDataBtn = document.getElementById('import-data');
const importFileInput = document.getElementById('import-file');

// DOM elements - Categories view
const categoriesView = document.getElementById('categories-view');
const categoriesList = document.getElementById('categories-list');
const noCategoriesEl = document.getElementById('no-categories');

// DOM elements - Category details view
const categoryDetailsView = document.getElementById('category-details-view');
const productsTable = document.getElementById('products-table');
const productsBody = document.getElementById('products-body');
const noProductsEl = document.getElementById('no-products');

// DOM elements - Fixed action buttons
const addCategoryBtn = document.getElementById('add-category-btn');
const addProductBtn = document.getElementById('add-product-btn');

// DOM elements - Add Category Modal
const addCategoryModal = document.getElementById('add-category-modal');
const modalCategoryName = document.getElementById('modal-category-name');
const modalCategoryBgColor = document.getElementById('modal-category-bg-color');
const modalCategoryTextColor = document.getElementById('modal-category-text-color');
const modalSaveCategory = document.getElementById('modal-save-category');
const modalCancelCategory = document.getElementById('modal-cancel-category');

// DOM elements - Add Product Modal
const addProductModal = document.getElementById('add-product-modal');
const modalProductName = document.getElementById('modal-product-name');
const modalProductPrice = document.getElementById('modal-product-price');
const modalProductPriceSYP = document.getElementById('modal-product-price-syp');
const modalProductBgColor = document.getElementById('modal-product-bg-color');
const modalProductTextColor = document.getElementById('modal-product-text-color');
const modalSaveProduct = document.getElementById('modal-save-product');
const modalCancelProduct = document.getElementById('modal-cancel-product');

// DOM elements - Edit Category Modal
const editCategoryModal = document.getElementById('edit-category-modal');
const modalEditCategoryName = document.getElementById('modal-edit-category-name');
const modalEditCategoryBgColor = document.getElementById('modal-edit-category-bg-color');
const modalEditCategoryTextColor = document.getElementById('modal-edit-category-text-color');
const modalDeleteCategory = document.getElementById('modal-delete-category');
const modalSaveEditCategory = document.getElementById('modal-save-edit-category');
const modalCancelEditCategory = document.getElementById('modal-cancel-edit-category');

// DOM elements - Edit Product Modal
const editProductModal = document.getElementById('edit-product-modal');
const modalEditProductName = document.getElementById('modal-edit-product-name');
const modalEditProductPrice = document.getElementById('modal-edit-product-price');
const modalEditProductPriceSYP = document.getElementById('modal-edit-product-price-syp');
const modalEditProductBgColor = document.getElementById('modal-edit-product-bg-color');
const modalEditProductTextColor = document.getElementById('modal-edit-product-text-color');
const modalDeleteProduct = document.getElementById('modal-delete-product');
const modalSaveEditProduct = document.getElementById('modal-save-edit-product');
const modalCancelEditProduct = document.getElementById('modal-cancel-edit-product');

// Load data from localStorage with improved error handling
function loadData() {
    try {
        console.log("بدء تحميل البيانات من التخزين المحلي...");
        
        const savedCategories = localStorage.getItem('app_categories');
        const savedProducts = localStorage.getItem('app_products');
        const savedDollarRate = localStorage.getItem('app_dollarRate');
        
        console.log("البيانات المحفوظة:", { 
            savedCategories: savedCategories ? "data found" : "no data", 
            savedProducts: savedProducts ? "data found" : "no data", 
            savedDollarRate: savedDollarRate 
        });
        
        // Initialize with empty arrays/objects if nothing is saved
        categories = [];
        products = {};
        
        if (savedCategories) {
            try {
                categories = JSON.parse(savedCategories);
                console.log("تم تحميل الأقسام:", categories.length);
            } catch (e) {
                console.error("خطأ في تحليل بيانات الأقسام:", e);
                categories = [];
            }
        }
        
        if (savedProducts) {
            try {
                products = JSON.parse(savedProducts);
                console.log("تم تحميل المنتجات");
            } catch (e) {
                console.error("خطأ في تحليل بيانات المنتجات:", e);
                products = {};
            }
        }
        
        if (savedDollarRate) {
            dollarRate = parseFloat(savedDollarRate) || 13000;
            dollarRateInput.value = dollarRate;
            console.log("تم تحميل سعر الدولار:", dollarRate);
        }
        
        dataChanged = false;
        updateSaveButtonStatus();
        
    } catch (error) {
        console.error("خطأ في تحميل البيانات:", error);
        // Reset to defaults on error
        categories = [];
        products = {};
        dollarRate = 13000;
        dollarRateInput.value = dollarRate;
    }
}

// Save data to localStorage with improved error handling and split storage
function saveData() {
    try {
        console.log("محاولة حفظ البيانات...");
        
        // حفظ كل مجموعة بيانات بشكل منفصل
        localStorage.setItem('app_categories', JSON.stringify(categories));
        localStorage.setItem('app_products', JSON.stringify(products));
        localStorage.setItem('app_dollarRate', dollarRate.toString());
        
        // التحقق من نجاح الحفظ
        const savedCategories = localStorage.getItem('app_categories');
        const savedProducts = localStorage.getItem('app_products');
        
        if (!savedCategories || !savedProducts) {
            throw new Error("فشل في التحقق من البيانات المحفوظة");
        }
        
        // إذا وصلنا إلى هنا، فقد نجح الحفظ
        dataChanged = false;
        updateSaveButtonStatus();
        console.log("تم حفظ البيانات بنجاح");
        
        return true;
    } catch (error) {
        console.error("خطأ في حفظ البيانات:", error);
        
        // Try to diagnose the issue
        try {
            // Check storage usage
            const storageInfo = calculateStorageUsage();
            console.log("معلومات التخزين:", storageInfo);
            
            if (storageInfo.percentUsed > 90) {
                alert("تحذير: مساحة التخزين المحلي ممتلئة تقريبًا. قم بتصدير بياناتك ثم حذف بعض العناصر.");
            } else {
                alert("حدث خطأ أثناء حفظ البيانات: " + error.message);
            }
        } catch (diagError) {
            alert("حدث خطأ أثناء حفظ البيانات. قد تفقد التغييرات عند تحديث الصفحة.");
        }
        
        return false;
    }
}

// Calculate storage usage for diagnostics
function calculateStorageUsage() {
    let total = 0;
    let used = 0;
    
    // Estimate localStorage size (depends on browser)
    const estimatedMax = 5 * 1024 * 1024; // ~5MB is commonly the limit
    
    // Calculate used size
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);
        used += (key.length + value.length) * 2; // UTF-16 characters are 2 bytes each
    }
    
    return {
        used: used,
        max: estimatedMax,
        percentUsed: (used / estimatedMax) * 100
    };
}

// Update save button to reflect data status
function updateSaveButtonStatus() {
    if (dataChanged) {
        saveDataBtn.classList.add('animate-bounce');
        saveDataBtn.classList.remove('bg-green-500', 'hover:bg-green-600');
        saveDataBtn.classList.add('bg-yellow-500', 'hover:bg-yellow-600');
        saveDataBtn.querySelector('svg').classList.add('animate-spin');
    } else {
        saveDataBtn.classList.remove('animate-bounce');
        saveDataBtn.classList.remove('bg-yellow-500', 'hover:bg-yellow-600');
        saveDataBtn.classList.add('bg-green-500', 'hover:bg-green-600');
        saveDataBtn.querySelector('svg').classList.remove('animate-spin');
    }
}

// Mark data as changed and needing save
function markDataChanged() {
    dataChanged = true;
    updateSaveButtonStatus();
}

// Export data as JSON file
exportDataBtn.addEventListener('click', function() {
    try {
        // Create data object with all app data
        const exportData = {
            categories: categories,
            products: products,
            dollarRate: dollarRate,
            exportDate: new Date().toISOString(),
            version: '1.0.0'
        };
        
        // Convert to JSON string
        const jsonData = JSON.stringify(exportData, null, 2);
        
        // Create a blob and download link
        const blob = new Blob([jsonData], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        
        // Create download link and trigger click
        const a = document.createElement('a');
        a.href = url;
        a.download = `product-manager-export-${new Date().toISOString().split('T')[0]}.json`;
        document.body.appendChild(a);
        a.click();
        
        // Clean up
        setTimeout(() => {
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }, 100);
        
        alert('تم تصدير البيانات بنجاح');
    } catch (error) {
        console.error('خطأ في تصدير البيانات:', error);
        alert('حدث خطأ أثناء تصدير البيانات');
    }
});

// Import data from JSON file
importDataBtn.addEventListener('click', function() {
    importFileInput.click();
});

importFileInput.addEventListener('change', function(e) {
    if (!e.target.files.length) return;
    
    const file = e.target.files[0];
    const reader = new FileReader();
    
    reader.onload = function(event) {
        try {
            const importedData = JSON.parse(event.target.result);
            
            // Validate imported data structure
            if (!importedData.categories || !importedData.products || importedData.dollarRate === undefined) {
                throw new Error('بنية البيانات المستوردة غير صالحة');
            }
            
            if (confirm('سيتم استبدال جميع البيانات الحالية بالبيانات المستوردة. هل أنت متأكد؟')) {
                // Import the data
                categories = importedData.categories;
                products = importedData.products;
                dollarRate = parseFloat(importedData.dollarRate);
                dollarRateInput.value = dollarRate;
                
                saveData();
                renderCategories();
                
                if (currentCategory) {
                    // If we're in a category view, re-render that too
                    if (products[currentCategory]) {
                        renderProducts(currentCategory);
                    } else {
                        // If the current category doesn't exist anymore, go back to home
                        showCategoriesView();
                    }
                }
                
                alert('تم استيراد البيانات بنجاح');
            }
        } catch (error) {
            console.error('خطأ في استيراد البيانات:', error);
            alert('حدث خطأ أثناء استيراد البيانات. تأكد من اختيار ملف صالح.');
        }
        
        // Reset file input
        e.target.value = '';
    };
    
    reader.readAsText(file);
});

// Update dollar rate
dollarRateInput.addEventListener('input', function() {
    dollarRate = parseFloat(this.value) || 0;
    markDataChanged();
    
    // Update prices in modals
    updateModalProductPriceSYP();
    updateModalEditProductPriceSYP();
    
    // Update products list if in category view
    if (currentCategory !== null) {
        renderProducts(currentCategory);
    }
});

// Explicitly save data when button clicked
saveDataBtn.addEventListener('click', function() {
    if (saveData()) {
        // Show success message
        const originalText = this.innerHTML;
        this.innerHTML = `
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 ml-1" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
            </svg>
            تم الحفظ
        `;
        
        setTimeout(() => {
            this.innerHTML = originalText;
        }, 2000);
    }
});

// Search functionality
searchInput.addEventListener('input', function() {
    searchTerm = this.value.trim().toLowerCase();
    
    if (searchTerm) {
        clearSearchBtn.classList.remove('hidden');
        searchResultsCount.classList.remove('hidden');
    } else {
        clearSearchBtn.classList.add('hidden');
        searchResultsCount.classList.add('hidden');
    }
    
    // Perform search based on current view
    if (currentCategory === null) {
        // Search in categories view
        renderCategories();
    } else {
        // Search in products view
        renderProducts(currentCategory);
    }
});

// Clear search
clearSearchBtn.addEventListener('click', function() {
    searchInput.value = '';
    searchTerm = '';
    clearSearchBtn.classList.add('hidden');
    searchResultsCount.classList.add('hidden');
    
    // Clear search and re-render current view
    if (currentCategory === null) {
        renderCategories();
    } else {
        renderProducts(currentCategory);
    }
});

// Highlight search matches in text
function highlightText(text, searchTerm) {
    if (!searchTerm) return text;
    
    const pattern = new RegExp(`(${escapeRegExp(searchTerm)})`, 'gi');
    return text.replace(pattern, '<span class="highlight">$1</span>');
}

// Escape regular expression special characters
function escapeRegExp(string) {
    return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Toggle action buttons based on current view
function updateActionButtons() {
    if (currentCategory === null) {
        // In categories view, show add category button, hide add product button
        addCategoryBtn.classList.remove('hidden');
        addProductBtn.classList.add('hidden');
    } else {
        // In products view, show add product button, hide add category button
        addCategoryBtn.classList.add('hidden');
        addProductBtn.classList.remove('hidden');
    }
}

// Open add category modal
addCategoryBtn.addEventListener('click', function() {
    showModal(addCategoryModal);
    modalCategoryName.focus();
});

// Open add product modal
addProductBtn.addEventListener('click', function() {
    if (!currentCategory) {
        alert('الرجاء اختيار قسم أولاً لإضافة منتج إليه');
        return;
    }
    
    showModal(addProductModal);
    modalProductName.focus();
});

// Update SYP price in add product modal
modalProductPrice.addEventListener('input', updateModalProductPriceSYP);

function updateModalProductPriceSYP() {
    const productPrice = parseFloat(modalProductPrice.value) || 0;
    const sypPrice = productPrice * dollarRate;
    modalProductPriceSYP.value = sypPrice > 0 ? sypPrice.toLocaleString() + ' ل.س' : '';
}

// Update SYP price in edit product modal
modalEditProductPrice.addEventListener('input', updateModalEditProductPriceSYP);

function updateModalEditProductPriceSYP() {
    const productPrice = parseFloat(modalEditProductPrice.value) || 0;
    const sypPrice = productPrice * dollarRate;
    modalEditProductPriceSYP.value = sypPrice > 0 ? sypPrice.toLocaleString() + ' ل.س' : '';
}

// Save category from modal
modalSaveCategory.addEventListener('click', function() {
    const categoryName = modalCategoryName.value.trim();
    const bgColor = modalCategoryBgColor.value;
    const textColor = modalCategoryTextColor.value;
    
    if (!categoryName) {
        alert('الرجاء إدخال اسم القسم');
        return;
    }
    
    // Check if category already exists
    if (categories.some(cat => cat.name === categoryName)) {
        alert('هذا القسم موجود بالفعل');
        return;
    }
    
    try {
        console.log("إضافة قسم جديد:", categoryName);
        
        const newCategory = {
            id: Date.now().toString(), // Use timestamp as unique ID
            name: categoryName,
            bgColor: bgColor,
            textColor: textColor,
            created: new Date().toISOString()
        };
        
        // Make sure categories is initialized as an array
        if (!Array.isArray(categories)) {
            console.error("خطأ: categories ليست مصفوفة!", categories);
            categories = [];
        }
        
        categories.push(newCategory);
        
        // Initialize empty products array for this category
        products[newCategory.id] = [];
        
        modalCategoryName.value = '';
        markDataChanged();
        renderCategories();
        
        hideModal(addCategoryModal);
        
        // Show confirmation
        alert(`تم إضافة القسم "${categoryName}" بنجاح`);
        
    } catch (error) {
        console.error("خطأ في إضافة القسم:", error);
        alert("حدث خطأ أثناء إضافة القسم. يرجى المحاولة مرة أخرى.");
    }
});

// Cancel category modal
modalCancelCategory.addEventListener('click', function() {
    modalCategoryName.value = '';
    hideModal(addCategoryModal);
});

// Save product from modal
modalSaveProduct.addEventListener('click', function() {
    if (!currentCategory) return;
    
    const productName = modalProductName.value.trim();
    const productPrice = parseFloat(modalProductPrice.value);
    const bgColor = modalProductBgColor.value;
    const textColor = modalProductTextColor.value;
    
    if (!productName) {
        alert('الرجاء إدخال اسم المنتج');
        return;
    }
    
    if (isNaN(productPrice) || productPrice <= 0) {
        alert('الرجاء إدخال سعر صحيح للمنتج');
        return;
    }
    
    const newProduct = {
        id: Date.now().toString(),
        name: productName,
        price: productPrice,
        bgColor: bgColor,
        textColor: textColor,
        created: new Date().toISOString()
    };
    
    // Add product to current category
    products[currentCategory].push(newProduct);
    
    // Clear inputs
    modalProductName.value = '';
    modalProductPrice.value = '';
    modalProductPriceSYP.value = '';
    
    markDataChanged();
    renderProducts(currentCategory);
    
    hideModal(addProductModal);
    
    // Show confirmation
    alert(`تم إضافة المنتج "${productName}" بنجاح`);
});

// Cancel product modal
modalCancelProduct.addEventListener('click', function() {
    modalProductName.value = '';
    modalProductPrice.value = '';
    modalProductPriceSYP.value = '';
    hideModal(addProductModal);
});

// Edit category name
editCategoryNameBtn.addEventListener('click', function() {
    if (!currentCategory) return;
    
    const category = categories.find(cat => cat.id === currentCategory);
    if (!category) return;
    
    modalEditCategoryName.value = category.name;
    modalEditCategoryBgColor.value = category.bgColor || '#5D5CDE';
    modalEditCategoryTextColor.value = category.textColor || '#FFFFFF';
    
    showModal(editCategoryModal);
    modalEditCategoryName.focus();
});

// Save edited category name
modalSaveEditCategory.addEventListener('click', function() {
    if (!currentCategory) return;
    
    const newName = modalEditCategoryName.value.trim();
    const newBgColor = modalEditCategoryBgColor.value;
    const newTextColor = modalEditCategoryTextColor.value;
    
    if (!newName) {
        alert('الرجاء إدخال اسم القسم');
        return;
    }
    
    // Check if new name already exists for a different category
    if (categories.some(cat => cat.name === newName && cat.id !== currentCategory)) {
        alert('هذا الاسم موجود بالفعل لقسم آخر');
        return;
    }
    
    // Update category name
    const category = categories.find(cat => cat.id === currentCategory);
    if (category) {
        category.name = newName;
        category.bgColor = newBgColor;
        category.textColor = newTextColor;
        
        categoryNameDisplay.textContent = newName;
        markDataChanged();
        renderCategories();
        
        // Update header
        categoryNameDisplay.textContent = newName;
        
        hideModal(editCategoryModal);
        
        // Show confirmation
        alert('تم تعديل القسم بنجاح');
    }
});

// Delete category from edit modal
modalDeleteCategory.addEventListener('click', function() {
    if (!currentCategory) return;
    
    deleteCategory(currentCategory);
    hideModal(editCategoryModal);
});

// Cancel edit category modal
modalCancelEditCategory.addEventListener('click', function() {
    hideModal(editCategoryModal);
});

// Edit product (opens edit modal)
function editProduct(productId) {
    if (!currentCategory) return;
    
    const categoryProducts = products[currentCategory] || [];
    const product = categoryProducts.find(p => p.id === productId);
    
    if (!product) return;
    
    // Store current editing product
    currentEditingProduct = productId;
    
    // Populate form
    modalEditProductName.value = product.name;
    modalEditProductPrice.value = product.price;
    modalEditProductBgColor.value = product.bgColor || '#FFFFFF';
    modalEditProductTextColor.value = product.textColor || '#000000';
    updateModalEditProductPriceSYP();
    
    // Show modal
    showModal(editProductModal);
    modalEditProductName.focus();
}

// Save edited product
modalSaveEditProduct.addEventListener('click', function() {
    if (!currentCategory || !currentEditingProduct) return;
    
    const newName = modalEditProductName.value.trim();
    const newPrice = parseFloat(modalEditProductPrice.value);
    const newBgColor = modalEditProductBgColor.value;
    const newTextColor = modalEditProductTextColor.value;
    
    if (!newName) {
        alert('الرجاء إدخال اسم المنتج');
        return;
    }
    
    if (isNaN(newPrice) || newPrice <= 0) {
        alert('الرجاء إدخال سعر صحيح للمنتج');
        return;
    }
    
    // Find and update product
    const categoryProducts = products[currentCategory] || [];
    const product = categoryProducts.find(p => p.id === currentEditingProduct);
    
    if (product) {
        product.name = newName;
        product.price = newPrice;
        product.bgColor = newBgColor;
        product.textColor = newTextColor;
        
        markDataChanged();
        renderProducts(currentCategory);
        
        hideModal(editProductModal);
        currentEditingProduct = null;
        
        // Show confirmation
        alert('تم تعديل المنتج بنجاح');
    }
});

// Delete product from edit modal
modalDeleteProduct.addEventListener('click', function() {
    if (!currentCategory || !currentEditingProduct) return;
    
    deleteProduct(currentEditingProduct);
    hideModal(editProductModal);
    currentEditingProduct = null;
});

// Cancel edit product modal
modalCancelEditProduct.addEventListener('click', function() {
    hideModal(editProductModal);
    currentEditingProduct = null;
});

// Delete a product
function deleteProduct(productId) {
    if (!currentCategory) return;
    
    if (confirm('هل أنت متأكد من حذف هذا المنتج؟')) {
        products[currentCategory] = products[currentCategory].filter(product => product.id !== productId);
        markDataChanged();
        renderProducts(currentCategory);
    }
}

// Delete a category
function deleteCategory(categoryId) {
    if (confirm('هل أنت متأكد من حذف هذا القسم وجميع منتجاته؟')) {
        categories = categories.filter(category => category.id !== categoryId);
        
        // Delete associated products
        delete products[categoryId];
        
        markDataChanged();
        renderCategories();
        
        // If we're deleting the current category, go back to home
        if (currentCategory === categoryId) {
            showCategoriesView();
        }
    }
}

// Navigate to home (categories view)
homeBtn.addEventListener('click', function() {
    showCategoriesView();
});

// Show category details (navigate to category)
function showCategoryDetails(categoryId) {
    currentCategory = categoryId;
    
    // Find category object
    const category = categories.find(cat => cat.id === categoryId);
    if (!category) return;
    
    // Update UI
    categoryNameDisplay.textContent = category.name;
    
    // Show breadcrumb
    categoryBreadcrumb.classList.remove('hidden');
    
    // Switch view
    showView(categoryDetailsView);
    
    // Update action buttons
    updateActionButtons();
    
    // Render products
    renderProducts(categoryId);
}

// Render categories in the home view
function renderCategories() {
    // Check if there are categories
    if (categories.length === 0) {
        noCategoriesEl.classList.remove('hidden');
        categoriesList.innerHTML = '';
        searchResultsCount.textContent = 'لا توجد أقسام';
        return;
    }
    
    // Sort categories by creation date (oldest first - so new categories appear at the end)
    const sortedCategories = [...categories].sort((a, b) => 
        new Date(a.created) - new Date(b.created)
    );
    
    // Filter categories if search term exists
    let filteredCategories = sortedCategories;
    if (searchTerm) {
        filteredCategories = sortedCategories.filter(category => 
            category.name.toLowerCase().includes(searchTerm)
        );
        
        // Update search results count
        searchResultsCount.textContent = `تم العثور على ${filteredCategories.length} قسم`;
    }
    
    // Hide "no categories" message if we have categories to show
    noCategoriesEl.classList.toggle('hidden', filteredCategories.length > 0);
    categoriesList.innerHTML = '';
    
    // If no search results, show a message
    if (searchTerm && filteredCategories.length === 0) {
        const noResults = document.createElement('div');
        noResults.className = 'text-center text-gray-500 dark:text-gray-400 p-8 col-span-full';
        noResults.textContent = 'لا توجد نتائج تطابق بحثك';
        categoriesList.appendChild(noResults);
        return;
    }
    
    filteredCategories.forEach(category => {
        const productCount = products[category.id]?.length || 0;
        const bgColor = category.bgColor || '#5D5CDE';
        const textColor = category.textColor || '#FFFFFF';
        
        const categoryCard = document.createElement('div');
        categoryCard.className = 'rounded-lg shadow p-4 hover:shadow-md transition-shadow cursor-pointer flex flex-col';
        categoryCard.style.backgroundColor = bgColor;
        categoryCard.style.color = textColor;
        
        // Highlight category name if search is active
        const categoryNameHtml = searchTerm ? 
            highlightText(category.name, searchTerm) : 
            category.name;
        
        categoryCard.innerHTML = `
            <div class="flex justify-between items-start mb-3">
                <h3 class="text-lg font-bold">${categoryNameHtml}</h3>
                <button class="mr-2 edit-cat-btn" style="color: ${textColor}">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
                    </svg>
                </button>
            </div>
            <div class="mt-auto">
                <span class="text-sm">
                    <span class="font-bold">${productCount}</span> منتج
                </span>
            </div>
        `;
        
        // Add click event to navigate to category details
        categoryCard.addEventListener('click', (e) => {
            // Don't navigate if clicking the edit button
            if (!e.target.closest('.edit-cat-btn')) {
                showCategoryDetails(category.id);
            }
        });
        
        // Add edit event
        const editBtn = categoryCard.querySelector('.edit-cat-btn');
        editBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            
            // Set current category and show edit modal
            currentCategory = category.id;
            modalEditCategoryName.value = category.name;
            modalEditCategoryBgColor.value = category.bgColor || '#5D5CDE';
            modalEditCategoryTextColor.value = category.textColor || '#FFFFFF';
            showModal(editCategoryModal);
            modalEditCategoryName.focus();
        });
        
        categoriesList.appendChild(categoryCard);
    });
}

// Render products for a specific category
function renderProducts(categoryId) {
    const categoryProducts = products[categoryId] || [];
    
    // Check if there are products
    if (categoryProducts.length === 0) {
        noProductsEl.classList.remove('hidden');
        productsTable.classList.add('hidden');
        searchResultsCount.textContent = 'لا توجد منتجات';
        return;
    }
    
    // Sort products by creation date (oldest first - so new products appear at the end)
    const sortedProducts = [...categoryProducts].sort((a, b) => 
        new Date(a.created) - new Date(b.created)
    );
    
    // Filter products if search term exists
    let filteredProducts = sortedProducts;
    if (searchTerm) {
        filteredProducts = sortedProducts.filter(product => 
            product.name.toLowerCase().includes(searchTerm) || 
            product.price.toString().includes(searchTerm)
        );
        
        // Update search results count
        searchResultsCount.textContent = `تم العثور على ${filteredProducts.length} منتج`;
    }
    
    // Hide "no products" message if we have products to show
    noProductsEl.classList.toggle('hidden', filteredProducts.length > 0);
    productsTable.classList.toggle('hidden', filteredProducts.length === 0);
    productsBody.innerHTML = '';
    
    // If we have search term but no results, show a message
    if (searchTerm && filteredProducts.length === 0) {
        noProductsEl.classList.remove('hidden');
        productsTable.classList.add('hidden');
        noProductsEl.textContent = 'لا توجد منتجات تطابق بحثك';
        return;
    }
    
    filteredProducts.forEach(product => {
        const sypPrice = product.price * dollarRate;
        const bgColor = product.bgColor || '#FFFFFF';
        const textColor = product.textColor || '#000000';
        
        const tr = document.createElement('tr');
        tr.className = 'border-b border-gray-200 dark:border-gray-700';
        tr.style.backgroundColor = bgColor;
        tr.style.color = textColor;
        
        // Highlight product name if search is active
        const productNameHtml = searchTerm ? 
            highlightText(product.name, searchTerm) : 
            product.name;
        
        // Highlight price if it matches search
        const productPriceHtml = searchTerm && product.price.toString().includes(searchTerm) ? 
            highlightText(product.price.toFixed(2), searchTerm) : 
            product.price.toFixed(2);
        
        tr.innerHTML = `
            <td class="p-3">
                <div class="flex items-center">
                    <span>${productNameHtml}</span>
                    <button class="mr-2 edit-product-btn">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
                        </svg>
                    </button>
                </div>
            </td>
            <td class="p-3">${productPriceHtml} $</td>
            <td class="p-3">${sypPrice.toLocaleString()} ل.س</td>
            <td class="p-3 text-center">
                <button class="view-product-btn">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                    </svg>
                </button>
            </td>
        `;
        
        // Add edit product event
        const editBtn = tr.querySelector('.edit-product-btn');
        editBtn.addEventListener('click', () => {
            editProduct(product.id);
        });
        
        // Add view product event (could be expanded in the future)
        const viewBtn = tr.querySelector('.view-product-btn');
        viewBtn.addEventListener('click', () => {
            editProduct(product.id);
        });
        
        productsBody.appendChild(tr);
    });
}

// Show categories view (home)
function showCategoriesView() {
    currentCategory = null;
    categoryBreadcrumb.classList.add('hidden');
    showView(categoriesView);
    updateActionButtons();
    
    // Re-render categories with current search term
    renderCategories();
}

// Show a modal dialog
function showModal(modal) {
    modal.classList.remove('hidden');
    
    // Activate modal elements with animation
    setTimeout(() => {
        modal.querySelector('.modal-overlay').classList.add('active');
        modal.querySelector('.modal-container').classList.add('active');
    }, 10);
    
    // Add event listeners to close modal when clicking outside
    modal.querySelector('.modal-overlay').addEventListener('click', function() {
        hideModal(modal);
    });
}

// Hide a modal dialog
function hideModal(modal) {
    modal.querySelector('.modal-overlay').classList.remove('active');
    modal.querySelector('.modal-container').classList.remove('active');
    
    // Wait for animation to finish before hiding
    setTimeout(() => {
        modal.classList.add('hidden');
    }, 300);
}

// Switch between views
function showView(viewToShow) {
    const views = document.querySelectorAll('.view-transition');
    
    views.forEach(view => {
        if (view === viewToShow) {
            view.classList.remove('hidden');
            // Trigger animation
            setTimeout(() => {
                view.classList.add('active');
            }, 50);
        } else {
            view.classList.remove('active');
            // Wait for animation to finish before hiding
            setTimeout(() => {
                view.classList.add('hidden');
            }, 300);
        }
    });
}

// Check if localStorage is available
function checkStorage() {
    try {
        const testKey = 'test_localStorage';
        localStorage.setItem(testKey, 'test');
        
        if (localStorage.getItem(testKey) === 'test') {
            localStorage.removeItem(testKey);
            return true;
        }
        return false;
    } catch (e) {
        return false;
    }
}

// Reset app data (for debugging)
function resetApp() {
    if (confirm('سيتم حذف جميع البيانات. هل أنت متأكد؟')) {
        localStorage.removeItem('app_categories');
        localStorage.removeItem('app_products');
        localStorage.removeItem('app_dollarRate');
        
        categories = [];
        products = {};
        dollarRate = 13000;
        dollarRateInput.value = dollarRate;
        
        renderCategories();
        alert('تم إعادة ضبط التطبيق بنجاح');
    }
}

// Set up auto-save feature
let autoSaveTimer;
function setupAutoSave() {
    // Clear any existing timer
    if (autoSaveTimer) {
        clearInterval(autoSaveTimer);
    }
    
    // Set up new timer to save every 3 minutes if changes exist
    autoSaveTimer = setInterval(() => {
        if (dataChanged) {
            console.log('Auto-saving data...');
            saveData();
        }
    }, 3 * 60 * 1000); // 3 minutes
}

// Initialize the app
function init() {
    // First check if localStorage is available
    if (!checkStorage()) {
        alert('التخزين المحلي غير متاح في المتصفح. لن يتم حفظ البيانات بين الجلسات.');
    }
    
    try {
        loadData();
        renderCategories();
        updateActionButtons();
        setupAutoSave();
        
        // Make functions available globally for event handlers
        window.deleteProduct = deleteProduct;
        window.deleteCategory = deleteCategory;
        window.showCategoryDetails = showCategoryDetails;
        window.resetApp = resetApp;  // For debugging
        
        // Debug info
        console.log('App initialized successfully!');
        console.log('Categories loaded:', categories.length);
        console.log('Dollar rate:', dollarRate);
        
        // For easy debugging in console
        console.log('DEBUG: You can use resetApp() to clear all data');
    } catch (error) {
        console.error('خطأ في تهيئة التطبيق:', error);
        alert('حدث خطأ أثناء تحميل التطبيق. سيتم إعادة تهيئته.');
        
        // Reset to defaults
        categories = [];
        products = {};
        dollarRate = 13000;
        dollarRateInput.value = dollarRate;
        renderCategories();
    }
}

// Start the app
init();
