# DuurzameScanner

## Overview

DuurzameScanner is a web application that allows users to scan products in a supermarket and view detailed information about them, including allergy information, sustainability details, and alternative products. Users can also finalize their order, which triggers a sound notification.

## Features

- Fetch and display all scanned products
- Display detailed information about a specific product in a popup
- Play a sound notification when a new product is added
- Play a sound notification when the order is finalized

## Functions

### `fetchAllScannedProducts`

Fetches all scanned products from the API and displays them in the product list.

```javascript
async function fetchAllScannedProducts() {
  try {
    console.log("Fetching all scanned products...");
    const response = await fetch(
      "https://nickvanhooff.com/DuurzameScannerApi/public/api/products-orders"
    );
    const productsOrders = await response.json();
    console.log("Products Orders:", productsOrders);
    const productListElement = document.getElementById("product-list");
    productListElement.innerHTML = ""; // Clear existing content
    if (productsOrders.length > previousProductCount) {
      document.getElementById("notification-sound").play();
    }
    previousProductCount = productsOrders.length;
    if (productsOrders.length > 0) {
      productsOrders.forEach((productOrder) => {
        const listItem = document.createElement("div");
        listItem.classList.add(
          "product-card",
          "bg-white",
          "p-4",
          "rounded-lg",
          "shadow-md",
          "cursor-pointer"
        );
        const productImage =
          productOrder.product.image || placeholderImage;
        listItem.innerHTML = `
        <div class="flex justify-between items-center">
          <img src="${productImage}" alt="${productOrder.name}" class="product-image" />
          <h3 class="text-lg font-semibold">${productOrder.name}</h3>
        </div>
        `;
        listItem.onclick = () =>
          displayProductDetails(productOrder.product.id);
        productListElement.appendChild(listItem);
      });
    }
  } catch (error) {
    console.error("Error fetching the scanned products:", error);
  }
}
```
### displayProductDetails
Fetches and displays the details of a specific product in a popup.

```javascript
async function displayProductDetails(productId) {
  try {
    console.log(`Fetching details for product ID: ${productId}`);
    const response = await fetch(
      `https://nickvanhooff.com/DuurzameScannerApi/public/api/product/${productId}`
    );
    const productOrder = await response.json();
    console.log("Product Order:", productOrder);
    const productName = productOrder.name;
    const productPrice = `€${productOrder.price}`;
    const productBrand = productOrder.brand.name;
    const productCategory = productOrder.categorie.name;
    const sustainabilities = productOrder.sustainabilities
      .map(
        (s) => `
        <li>
          <strong>${s.label_name}</strong>: Eco Score - ${s.eco_score}, 
          Bio Certified - ${s.bio_certified ? "Yes" : "No"}, 
          Animal Friendly Score - ${s.animal_friendly_score}
        </li>
      `
      )
      .join("");
    const allergens = productOrder.allergens
      .map(
        (a) => `
        <li>
          <strong>${a.name}</strong>: ${a.description}
        </li>
      `
      )
      .join("");
    const alternatives = await Promise.all(
      productOrder.alternatives.map(async (alt) => {
        const alternativeProductId = alt.pivot.alternative_id;
        if (!alternativeProductId) {
          console.error("Alternative product ID is undefined:", alt);
          return "";
        }
        const altResponse = await fetch(
          `https://nickvanhooff.com/DuurzameScannerApi/public/api/product/${alternativeProductId}`
        );
        const altProduct = await altResponse.json();
        console.log("Alternative Product:", altProduct);
        const altImage = altProduct.image || placeholderImage;
        return `
        <div class="swiper-slide">
          <div class="card p-4 bg-white rounded shadow-md">
            <img src="${altImage}" alt="${
          altProduct.name
        }" class="w-full h-32 object-cover mb-2" />
            <p><strong>${altProduct.name}</strong></p>
            <p>Price: €${altProduct.price || "N/A"}</p>
            <p>${alt.pivot.reason || "N/A"}</p>
          </div>
        </div>
      `;
      })
    );
    document.getElementById("product-name").textContent = productName;
    document.getElementById("product-price").textContent = productPrice;
    document.getElementById("allergy-text").innerHTML =
      allergens || "No allergens";
    document.getElementById("sustainability-text").innerHTML =
      sustainabilities || "No sustainability information";
    document.getElementById("alternatives-text").innerHTML =
      alternatives.length > 0
        ? alternatives.join("")
        : "No alternatives available";
    document.getElementById("popup-overlay").classList.remove("hidden");
    setTimeout(() => {
      new Swiper(".swiper-container", {
        slidesPerView: 1.5,
        spaceBetween: 10,
        pagination: {
          el: ".swiper-pagination",
          clickable: true,
        },
        navigation: {
          nextEl: ".swiper-button-next",
          prevEl: ".swiper-button-prev",
        },
      });
    }, 100);
  } catch (error) {
    console.error("Error fetching the product details:", error);
  }
}
```

### `inalizeOrder
Finalizes the order by sending the order data to the API and plays the kaching sound.
    
```javascript
    async function finalizeOrder(orderData) {
      try {
        const response = await fetch(
          "https://nickvanhooff.com/DuurzameScannerApi/public/api/finalize-order",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(orderData),
          }
        );
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        const responseData = await response.json();
        console.log("Order finalized successfully:", responseData);
        document.getElementById("kaching-sound").play();
      } catch (error) {
        console.error("There was a problem with the fetch operation:", error);
      }
    }
```
### window.onload
Initializes the page by fetching all scanned products and setting an interval to fetch them every 3 seconds.
    
```javascript
    window.onload = () => {
      fetchAllScannedProducts();
      setInterval(fetchAllScannedProducts, 3000);
    };
```
### dependencies
Tailwind CSS
Swiper
Swiper JS