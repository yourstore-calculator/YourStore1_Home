# YourStore1_Home
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8" />
  <title>Advanced Price Calculator</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: ltr;
      padding: 30px;
    }
    select, input, textarea {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
      box-sizing: border-box;
      background-color: #333;
      color: white;
    }
    textarea {
        resize: vertical;
        min-height: 80px;
    }
    input:disabled {
      background-color: #555;
      cursor: not-allowed;
      color: #aaa;
    }
    .section {
      background-color: #252526;
      padding: 15px;
      border-radius: 8px;
      margin-bottom: 20px;
      border: 1px solid #333;
    }
    .addon-section {
        background-color: #2a2d2e;
        padding: 10px;
        border-radius: 6px;
        margin-top: 10px;
    }
    label {
        font-weight: bold;
        color: #00e676;
    }
    button {
      background-color: #007acc;
      color: white;
      padding: 12px 22px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #005a9e;
    }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }

    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      border-bottom: 1px solid #555;
      padding: 15px 0;
      line-height: 1.8;
    }
    .result-item:last-child { border-bottom: none; }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 15px;
      border-top: 2px solid #00e676;
      margin-top: 15px;
    }
    .result-controls { display: flex; gap: 10px; margin-top: 10px; align-items: center; }
    .result-controls input { width: 70px; text-align: center; }
    .result-controls label { font-size: 0.9em; }
    .delete-btn {
        background-color: #c0392b;
        color: white;
        border: none;
        padding: 5px 10px;
        border-radius: 4px;
        cursor: pointer;
        margin-left: auto; /* Pushes the button to the far right */
    }
    .delete-btn:hover { background-color: #a52f22; }

  </style>
</head>
<body>

<div class="section">
    <label for="customerName">Customer Name:</label>
    <input type="text" id="customerName" placeholder="e.g., John Doe" />
    <label for="customerPhone">Customer Phone:</label>
    <input type="text" id="customerPhone" placeholder="e.g., 99455082" />
</div>

<div class="section">
  <label for="mainCategory">Main Category:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">Sub Type:</label>
  <select id="subType" onchange="updateUIBasedOnType()"></select>
</div>

<div class="section">
  <label for="height">Height (meter):</label>
  <input type="number" id="height" step="0.01" value="1" />
  
  <label for="width">Width (meter):</label>
  <input type="number" id="width" step="0.01" value="1" />

  <label for="quantity">Quantity:</label>
  <input type="number" id="quantity" value="1" />
</div>

<div class="section" id="addonsHost"></div>

<div class="section">
    <label for="additionalDetails">Additional Details (Optional):</label>
    <textarea id="additionalDetails" placeholder="Enter any notes or additional terms to be shown in the quotation..."></textarea>
</div>


<button onclick="calculate()">➕ Add to Results</button>
<button onclick="clearAllResults()" class="btn-danger">🗑️ Clear All Results</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 Save as Word</button>

<div class="results" id="results"></div>

<script>
const SHIPPING_RATE = 48;

const productData = {
    "Windows": {
        "Window Double Glass Double Frame Fixed": { price: 34, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Double Glass Double Frame 1-Way": { price: 34, fixed_component_cost: 39, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Double Glass Double Frame 2-Way": { price: 34, fixed_component_cost: 39 + 19, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Double Glass Single Frame Fixed": { price: 26, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Double Glass Single Frame 1-Way": { price: 26, fixed_component_cost: 20, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Double Glass Single Frame 2-Way": { price: 26, fixed_component_cost: 20 + 12, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "Window Single Glass Single Frame Fixed": { price: 20, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "Window Single Glass Single Frame 1-Way": { price: 20, fixed_component_cost: 13, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "Window Single Glass Single Frame 2-Way": { price: 20, fixed_component_cost: 13 + 4, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "Sliding Windows": { price: 41, fixed_component_cost: 10, cbm: 0.13, method: 'per_meter', addons: ['curtain'] },
        "Electric Windows": { price: 102, cbm: 0.13, method: 'per_meter' },
        "Skylight without Motor": { price: 56, cbm: 0.13, method: 'per_meter' },
        "Skylight with Motor": { price: 145, cbm: 0.13, method: 'per_meter' },
        "Heavy Curtain Wall": { price: 56, cbm: 0.15, method: 'per_meter' },
        "Light Curtain Wall": { price: 45, cbm: 0.15, method: 'per_meter' },
    },
    "Doors": {
        "Entrance Door - Zinc": { price: 66, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Stainless Steel": { price: 120, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Cast Aluminum": { price: 168, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "WPC Door": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC Door - with Wood": { price: 50, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC Door - with Soundproof Filling": { price: 60, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC Door - with Aluminum Frame": { price: 67, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "Aluminum Door": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - with Wood": { price: 75, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Full": { price: 85, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Hidden": { price: 110, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Exterior": { price: 61, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Bathroom Door - New Type": { price: 55, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Old Type": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Hidden Glass": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
    },
    "Sliding Doors": {
        "Interior Sliding Door - Glass": { price: 38, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "Interior Sliding Door - Solid": { price: 41, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 1 Panel Open": { price: 55, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 2 Panels Open": { price: 58, cbm: 0.15, method: 'per_meter' },
        "WPC Sliding Door": { price: 61, cbm: 0.15, method: 'per_meter' },
    },
    "Folding Doors": {
        "Interior Folding Door": { price: 39, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "Exterior Folding Door": { price: 56, cbm: 0.15, method: 'per_meter' },
    },
    "Exterior Shutters": {
        "Rolling Shutter": { price: 28, cbm: 0.20, method: 'per_meter' },
    },
    "Garden Gates": {
        "Cast Aluminum Garden Gate": { price: 91, cbm: 0.20, method: 'per_meter' },
    },
    "Barriers": {
        "Stair Barrier": { price: 43, cbm: 0.05, method: 'per_meter' },
        "Bathroom Barrier": { price: 32, cbm: 0.05, method: 'per_meter' },
    }
};

const addonPrices = {
    curtain: 26, net: { 'door': 39, 'folding': 18, 'sliding': 14 }, telbisa: 10
};

let resultsList = [];

function findProductData(subTypeName) {
    for (const category in productData) {
        if (productData[category][subTypeName]) {
            return productData[category][subTypeName];
        }
    }
    return null;
}

function initializeApp() {
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- Select Category --</option>`;
    Object.keys(productData).forEach(cat => mainCat.innerHTML += `<option value="${cat}">${cat}</option>`);
    loadSubTypes();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => subType.innerHTML += `<option value="${sub}">${sub}</option>`);
    }
    updateUIBasedOnType();
}

function updateUIBasedOnType() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    
    if (!mainCatVal || !subVal) {
        document.getElementById("addonsHost").innerHTML = "";
        return;
    }

    const data = productData[mainCatVal][subVal];
    const heightInput = document.getElementById("height"), widthInput = document.getElementById("width");
    const addonsHost = document.getElementById("addonsHost");
    addonsHost.innerHTML = ""; 

    heightInput.disabled = false;
    widthInput.disabled = false;

    if (data.method === 'per_unit') {
        heightInput.value = data.std_h;
        widthInput.value = data.std_w;
    }
    
    if (data.addons) {
        if (data.addons.includes('curtain')) addonsHost.innerHTML += `<div class="addon-section"><label><input type="checkbox" id="addCurtain" /> Add Internal Curtain (26 OMR per meter)</label></div>`;
        if (data.addons.includes('net')) addonsHost.innerHTML += `<div class="addon-section"><label for="netType">Add Net:</label><select id="netType"><option value="">-- None --</option><option value="door">Door (${addonPrices.net['door']} OMR per meter)</option><option value="folding">Folding (${addonPrices.net['folding']} OMR per meter)</option><option value="sliding">Sliding (${addonPrices.net['sliding']} OMR per meter)</option></select></div>`;
        if (data.addons.includes('telbisa')) {
             addonsHost.innerHTML += `<div class="addon-section"><label><input type="checkbox" id="addTelbisa" /> Add Ceiling Cladding</label><input type="number" id="telbisaLength" placeholder="Cladding length in meters" style="display:none;" value="1" step="0.1" /></div>`;
             document.getElementById('addTelbisa').onchange = e => document.getElementById('telbisaLength').style.display = e.target.checked ? 'block' : 'none';
        }
    }
}

function calculate() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    if (!mainCatVal || !subVal) { alert("Please select a category and sub-type."); return; }
    
    const newItem = {
        id: Date.now(),
        sub: subVal,
        h: parseFloat(document.getElementById("height").value) || 0,
        w: parseFloat(document.getElementById("width").value) || 0,
        qty: parseInt(document.getElementById("quantity").value) || 1,
        addons: {
            curtain: document.getElementById("addCurtain")?.checked || false,
            netType: document.getElementById("netType")?.value || null,
            telbisa: document.getElementById("addTelbisa")?.checked || false,
            telbisaLength: parseFloat(document.getElementById("telbisaLength")?.value) || 0
        },
        details: {},
        singleItemPrice: 0,
        final: 0
    };

    if (newItem.h === 0 || newItem.w === 0) { alert("Dimensions must be greater than zero."); return; }
    
    recalculateItemPrice(newItem);
    resultsList.push(newItem);
    renderResults();
}

function recalculateItemPrice(item) {
    const data = findProductData(item.sub);
    if (!data) return;

    const area = item.h * item.w;
    let basePrice = 0, sizePenalty = 0;

    if (data.method === 'per_meter') {
        basePrice = data.price * area;
    } else if (data.method === 'per_unit') {
        basePrice = data.price;
        const stdArea = data.std_h * data.std_w;
        if (area > stdArea) sizePenalty = Math.ceil((area - stdArea) / 0.1) * 2;
    }

    let itemPrice = basePrice + sizePenalty;
    let addonsDetails = `<li>${item.sub}</li>`;
    
    if (data.fixed_component_cost) {
        itemPrice += data.fixed_component_cost;
        addonsDetails += `<li>Additional Fixed Cost: ${data.fixed_component_cost.toFixed(2)}</li>`;
    }

    if (item.addons.curtain) {
        const cost = addonPrices.curtain * area; itemPrice += cost; addonsDetails += `<li>Add Curtain</li>`;
    }
    if (item.addons.netType) {
        const cost = (area * 0.5) * addonPrices.net[item.addons.netType]; itemPrice += cost; addonsDetails += `<li>Add Net (${item.addons.netType})</li>`;
    }
    if (item.addons.telbisa && item.addons.telbisaLength > 0) {
        const cost = Math.ceil(item.addons.telbisaLength) * addonPrices.telbisa; itemPrice += cost; addonsDetails += `<li>Ceiling Cladding (${item.addons.telbisaLength}m)</li>`;
    }
    if (data.special === 'add_10') { itemPrice += 10; addonsDetails += `<li>Special Addition: 10.00</li>`; }

    let shippingCBM = ((item.addons.telbisa && item.addons.telbisaLength > 0) ? ((item.h + item.addons.telbisaLength) * item.w) : area) * data.cbm;
    const shippingCost = shippingCBM * SHIPPING_RATE;

    item.singleItemPrice = itemPrice + shippingCost;
    item.final = item.singleItemPrice * item.qty;
    item.details = { basePrice, sizePenalty, addonsHTML: addonsDetails, shippingPerItem: shippingCost };
}

function updateItem(id, field, value) {
    const itemToUpdate = resultsList.find(item => item.id === id);
    if (!itemToUpdate) return;
    
    itemToUpdate[field] = (field === 'qty') ? parseInt(value) : parseFloat(value);
    
    recalculateItemPrice(itemToUpdate);
    renderResults();
}

function deleteItem(id) {
    resultsList = resultsList.filter(item => item.id !== id);
    renderResults();
}

function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    let sum = 0;

    resultsList.forEach(r => {
        sum += r.final;
        container.innerHTML += `
            <div class="result-item">
                <b>Type:</b> ${r.sub}<br>
                <b>Price Per Item: ${r.singleItemPrice.toFixed(2)} OMR</b> | 
                <b>Total Price: ${r.final.toFixed(2)} OMR</b>
                <div class="result-controls">
                    <label>Qty:</label>
                    <input type="number" value="${r.qty}" onchange="updateItem(${r.id}, 'qty', this.value)">
                    <label>H:</label>
                    <input type="number" step="0.01" value="${r.h}" onchange="updateItem(${r.id}, 'h', this.value)">
                    <label>W:</label>
                    <input type="number" step="0.01" value="${r.w}" onchange="updateItem(${r.id}, 'w', this.value)">
                    <button class="delete-btn" onclick="deleteItem(${r.id})">Delete 🗑️</button>
                </div>
            </div>`;
    });

    if (resultsList.length > 0) {
        const comm = sum * 0.04;
        const total = sum + comm;
        container.innerHTML += `
            <div class="summary">
                <div style="display: flex; justify-content: space-between; padding: 5px 0; font-size: 1.2em; color: white;">
                    <span>Subtotal:</span>
                    <span style="font-weight: bold;">${sum.toFixed(2)} OMR</span>
                </div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0; font-size: 1.2em; border-bottom: 1px solid #555; margin-bottom: 10px; color: white;">
                    <span>Office Commission (4%):</span>
                    <span style="font-weight: bold;">${comm.toFixed(2)} OMR</span>
                </div>
                <div style="background-color: #00e676; color: #1e1e1e; padding: 15px; border-radius: 5px; text-align: center; margin-top: 10px;">
                    <span style="font-size: 1.3em; font-weight: bold;">💰 Grand Total</span>
                    <div style="font-size: 2.0em; font-weight: bold; margin-top: 5px;">${total.toFixed(2)} OMR</div>
                </div>
            </div>`;
    }
}

function clearAllResults() {
    resultsList = [];
    document.getElementById("results").innerHTML = "";
    document.getElementById("mainCategory").value = "";
    document.getElementById("height").value = "1";
    document.getElementById("width").value = "1";
    document.getElementById("quantity").value = "1";
    document.getElementById("customerName").value = "";
    document.getElementById("customerPhone").value = "";
    document.getElementById("additionalDetails").value = "";
    loadSubTypes(); 
}

function formatNumber(num) {
    return parseFloat(num.toFixed(2));
}

function saveAsWord(){
    if (resultsList.length === 0) {
        alert("There are no results to save.");
        return;
    }

    const customerName = document.getElementById('customerName').value || 'Customer';
    const customerPhone = document.getElementById('customerPhone').value || 'N/A';
    const additionalDetails = document.getElementById('additionalDetails').value;

    const headerHtml = `
        <div style="width: 100%; font-family: Arial, sans-serif; text-align: center; margin-bottom: 30px; border-bottom: 2px solid #4472C4; padding-bottom: 20px;">
            <div style="font-size: 26px; font-weight: bold; color: #4472C4; margin-bottom: 15px;">
                BLUE WAVES SERVICES LLC
            </div>
            <table width="100%" style="font-family: Arial, sans-serif; font-size: 15px; color: #4472C4; border-collapse: collapse;">
                <tr>
                    <td style="text-align: left; width: 33%;">OMAN – MUSCAT</td>
                    <td style="text-align: center; width: 34%;">SR. NO. : 1595256</td>
                    <td style="text-align: right; width: 33%;">TEL: 77 22 45 11 – 90 99 88 10</td>
                </tr>
            </table>
        </div>`;

    const customerHtml = `
        <div style="background-color: #f2f2f2; border: 1px solid #ddd; color: #333; padding: 14px; font-family: Arial, sans-serif; font-size: 17px; margin-bottom: 25px; border-radius: 5px;">
            <b>Quotation for: </b> ${customerName} - ${customerPhone}
        </div>`;

    let tableHtml = `<table border="1" width="100%" style="border-collapse: collapse; font-family: Arial, sans-serif; text-align: center; font-size: 14px;">
                        <thead>
                            <tr style="background-color: #4472C4; color: white;">
                                <th style="padding: 10px;">#</th>
                                <th style="padding: 10px;">Type / Style</th>
                                <th style="padding: 10px;">Height</th>
                                <th style="padding: 10px;">Width</th>
                                <th style="padding: 10px;">Qty</th>
                                <th style="padding: 10px;">Additional Details</th>
                                <th style="padding: 10px;">Shipping Per Item</th>
                                <th style="padding: 10px;">Unit Price</th>
                                <th style="padding: 10px;">Total</th>
                            </tr>
                        </thead>
                        <tbody>`;
    
    let subtotal = 0;

    resultsList.forEach((r, index) => {
        subtotal += r.final;
        const descriptionCell = `<ul style="text-align: left; margin: 0; padding-left: 15px; list-style-position: inside;">${r.details.addonsHTML}</ul>`;
        
        tableHtml += `<tr>
                        <td style="padding: 10px; font-weight: bold;">${index + 1}</td>
                        <td style="padding: 10px; font-weight: bold;">${r.sub}</td>
                        <td style="padding: 10px;">${formatNumber(r.h)}m</td>
                        <td style="padding: 10px;">${formatNumber(r.w)}m</td>
                        <td style="padding: 10px;">${r.qty}</td>
                        <td style="padding: 10px; text-align: left;">${descriptionCell}</td>
                        <td style="padding: 10px; background-color: #f2f2f2;">${formatNumber(r.details.shippingPerItem)}</td>
                        <td style="padding: 10px; background-color: #f2f2f2;">${formatNumber(r.singleItemPrice)}</td>
                        <td style="padding: 10px; background-color: #f2f2f2; font-weight: bold;">${formatNumber(r.final)}</td>
                      </tr>`;
    });
    tableHtml += `</tbody></table>`;
    
    const commission = subtotal * 0.04;
    const grandTotal = subtotal + commission;

    const summaryTable = `
        <table width="45%" align="right" style="border-collapse: collapse; font-family: Arial, sans-serif; margin-top: 25px; font-size: 15px;">
            <tbody>
                <tr>
                    <td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Subtotal</td>
                    <td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(subtotal)} OMR</td>
                </tr>
                <tr>
                    <td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Office Commission (4%)</td>
                    <td style="padding: 14px; text-align: right; font-weight: bold; background-color: #f2f2f2; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(commission)} OMR</td>
                </tr>
                <tr style="background-color: #4472C4; color: white;">
                    <td style="padding: 16px; font-weight: bold; border: 1px solid #4472C4; font-size: 1.5em;">Grand Total</td>
                    <td style="padding: 16px; text-align: right; font-weight: bold; border: 1px solid #4472C4; font-size: 1.9em; vertical-align: middle;">${formatNumber(grandTotal)} OMR</td>
                </tr>
            </tbody>
        </table>`;
    
    let notesHtml = '';
    if (additionalDetails.trim() !== '') {
        notesHtml = `
        <div style="clear: both; padding-top: 20px;"></div>
        <div style="font-family: Arial, sans-serif; margin-top: 30px; border-top: 1px solid #ccc; padding-top: 15px;">
            <h3 style="font-size: 17px; color: #4472C4;">Notes:</h3>
            <p style="font-size: 15px; white-space: pre-wrap;">${additionalDetails}</p>
        </div>`;
    }

    const footerNotesHtml = `
        <div style="clear: both; font-family: Arial, sans-serif; margin-top: 50px; text-align: center; border-top: 2px solid #4472C4; padding-top: 15px; font-size: 16px; color: #333;">
            <p style="margin: 5px 0;"><b>*Price includes delivery*</b></p>
            <p style="margin: 5px 0;"><b>*Price does not include installation*</b></p>
        </div>`;

    const finalHtml = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
                        <head><meta charset='utf-8'><title>Quotation for - ${customerName}</title></head>
                        <body dir="ltr" style="padding: 20px;">` 
                      + headerHtml + customerHtml + tableHtml + summaryTable + notesHtml + footerNotesHtml
                      + `</body></html>`;

    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(finalHtml);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = `Quotation for - ${customerName}.doc`;
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

initializeApp();
</script>

</body>
</html>
