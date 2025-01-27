
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form id="recordForm">
        <label>Customer ID: <input type="number" id="customerId" required></label><br>
        <label>Date: <input type="date" id="date" required></label><br>
        <label>Quantity (L): <input type="number" id="quantity" required></label><br>
        <label>Fat (%): <input type="number" step="0.1" id="fat" required></label><br>
        <button type="submit">Add Record</button>
    </form>
    
    <hr>
    
    <form id="viewForm">
        <label>View Data for Customer ID: <input type="number" id="viewCustomerId" required></label>
        <button type="submit">View Data</button>
    </form>
    
    <div id="customerData">
        <h3>Customer Data</h3>
        <table border="1">
            <thead>
                <tr>
                    <th>Date</th>
                    <th>Quantity (L)</th>
                    <th>Fat (%)</th>
                </tr>
            </thead>
            <tbody id="dataBody"></tbody>
        </table>
    </div>
    
    
    <script>
       let db;

// Initialize IndexedDB
const request = indexedDB.open('MilkCenterDB', 1);

request.onupgradeneeded = function (event) {
    db = event.target.result;
    const customerStore = db.createObjectStore('customers', { keyPath: 'id' });
    customerStore.createIndex('name', 'name', { unique: false });
};

request.onsuccess = function (event) {
    db = event.target.result;
    console.log('Database initialized');
};

// Add Daily Record
function addDailyRecord(customerId, date, quantity, fat) {
    const transaction = db.transaction('customers', 'readwrite');
    const store = transaction.objectStore('customers');

    const request = store.get(customerId);
    request.onsuccess = function () {
        const customer = request.result || { id: customerId, records: [] };
        customer.records.push({ date, quantity, fat });
        store.put(customer);
        console.log(`Record added for Customer ${customerId}`);
    };
}

// Display Customer Data
function displayCustomerData(customerId) {
    const transaction = db.transaction('customers', 'readonly');
    const store = transaction.objectStore('customers');

    const request = store.get(customerId);
    request.onsuccess = function () {
        const customer = request.result;
        const dataBody = document.getElementById('dataBody');
        dataBody.innerHTML = '';

        if (customer && customer.records.length > 0) {
            customer.records.forEach(record => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${record.date}</td>
                    <td>${record.quantity}</td>
                    <td>${record.fat}</td>
                `;
                dataBody.appendChild(row);
            });
        } else {
            dataBody.innerHTML = '<tr><td colspan="3">No data found for this customer</td></tr>';
        }
    };
}

// Handle Form Submissions
document.getElementById('recordForm').addEventListener('submit', function (event) {
    event.preventDefault();
    const customerId = parseInt(document.getElementById('customerId').value);
    const date = document.getElementById('date').value;
    const quantity = parseFloat(document.getElementById('quantity').value);
    const fat = parseFloat(document.getElementById('fat').value);

    addDailyRecord(customerId, date, quantity, fat);
});

document.getElementById('viewForm').addEventListener('submit', function (event) {
    event.preventDefault();
    const customerId = parseInt(document.getElementById('viewCustomerId').value);
    displayCustomerData(customerId);
});

    </script>
    
</body>
</html>
