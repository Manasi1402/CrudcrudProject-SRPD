# CrudcrudProject-SRPD

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form onsubmit="saveToLocalStorage(event)">
        <label>Selling Price</label>
        <input type="text" name="amount" required/>
        <label>Product Name</label>
        <input type="text" name="product" required/>
        <button>Add Product</button>
        <br>
        <h3><b>Products</b></h3>
    </form>
    <ul id="listofitems"></ul>
    <h3 id="totalAmount">Total Amount: 0.00</h3>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/1.4.0/axios.min.js"></script>
    <script>
        let totalAmount = 0;
        let isEditing = false;
        function saveToLocalStorage(event) {
            event.preventDefault();
            const amount = event.target.amount.value;
            const product = event.target.product.value;
            const userDetails = {
                amount: amount,
                product: product,
            };
            axios.post("https://crudcrud.com/api/f92eb2ff0f3a43c7aec4f45f35e3c251/appoinmentData", userDetails)
                .then((response) => {
                    showUserOnScreen(response.data);
                    console.log(response);
                })
                .catch((err) => {
                    document.body.innerHTML = document.body.innerHTML;
                    console.log(err);
                });
            if (isEditing) {
                updateUserData(product, userDetails);
                isEditing = false;
            } else {
                addUserData(userDetails);
            }
            event.target.reset();
        }
        function addUserData(userDetails) {
            const storedUsers = JSON.parse(localStorage.getItem('users')) || [];
            storedUsers.push(userDetails);
            localStorage.setItem('users', JSON.stringify(storedUsers));
            localStorage.setItem(userDetails.product, JSON.stringify(userDetails));
            // Update the total amount
            totalAmount += parseFloat(userDetails.amount);
            // Display the total amount on the UI
            document.getElementById('totalAmount').textContent = 'Total Amount: ' + totalAmount.toFixed(2);
            showUserOnScreen(userDetails);
        }
        function updateUserData(product, userDetails) {
            const storedUsers = JSON.parse(localStorage.getItem('users')) || [];
            const updatedUsers = storedUsers.map(user => {
                if (user.product === product) {
                    return userDetails;
                }
                return user;
            });
        }
        function showUserOnScreen(user) {
            const parentElement = document.getElementById('listofitems');
            const listItem = document.createElement('li');
            listItem.setAttribute('data-email', user._id);
            listItem.textContent = user.amount + ' - ' + user._id + ' - ' + user.product;
            const deleteButton = document.createElement('button');
            deleteButton.textContent = 'Delete';
            deleteButton.addEventListener('click', function() {
                deleteUser(user._id);
            });
            listItem.appendChild(deleteButton);
            parentElement.appendChild(listItem);
            // Update the total amount on the UI
            document.getElementById('totalAmount').textContent = 'Total Amount: ' + totalAmount.toFixed(2);
        }
        function deleteUser(userId) {
            axios.delete(`https://crudcrud.com/api/f92eb2ff0f3a43c7aec4f45f35e3c251/appoinmentData/${userId}`)
                .then((response) => {
                    removeUserFromScreen(userId);
                    removeUserFromLocalStorage(userId);
                })
                .catch((err) => {
                    console.log(err);
                });
        }
        function removeUserFromScreen(userId) {
            const listItem = document.querySelector(`li[data-email="${userId}"]`);
            listItem.remove();
        }
        function removeUserFromLocalStorage(userId) {
            const storedUsers = JSON.parse(localStorage.getItem('users')) || [];
            const updatedUsers = storedUsers.filter(user => user._id !== userId);
            localStorage.setItem('users', JSON.stringify(updatedUsers));
            localStorage.removeItem(userId);
            // Recalculate the total amount after removing the user
            totalAmount = 0;
            updatedUsers.forEach(user => {
                totalAmount += parseFloat(user.amount);
            });
            // Display the updated total amount on the UI
            document.getElementById('totalAmount').textContent = 'Total Amount: ' + totalAmount.toFixed(2);
        }
        // Load existing users from local storage and display them on the UI
        window.addEventListener('DOMContentLoaded', function() {
            axios.get("https://crudcrud.com/api/f92eb2ff0f3a43c7aec4f45f35e3c251/appoinmentData")
                .then((response) => {
                    for (let i = 0; i < response.data.length; i++) {
                        showUserOnScreen(response.data[i]);
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        const storedUsers = JSON.parse(localStorage.getItem('users')) || [];
            storedUsers.forEach(function(user) {
                showUserOnScreen(user);
            });
        });
    </script>
</body>
</html>
