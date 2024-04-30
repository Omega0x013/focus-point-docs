Functions and Classes
---------

deadline.js
~~~~~~~~~~~~~~

saveDeadline()
**************

.. code-block:: javascript

   function saveDeadline() {
       // Get input values
       var name = document.getElementById("name").value;
       var description = document.getElementById("description").value;
       var date = document.getElementById("date").value;

       // Create a new reminder object
       var reminder = {
           name: name,
           description: description,
           date: date
       };

       // Save the reminder to local storage
       var reminders = JSON.parse(localStorage.getItem("reminders")) || [];
       reminders.push(reminder);
       localStorage.setItem("reminders", JSON.stringify(reminders));

       // Display reminders
       displayReminders();
   }

This function saves a deadline reminder entered by the user to the local storage. It retrieves the name, description, and date of the reminder from the HTML input fields, creates a JavaScript object representing the reminder, saves it to local storage, and then calls `displayReminders()` to update the displayed list of reminders.

displayReminders()
**************

.. code-block:: javascript

   function displayReminders() {
       var reminders = JSON.parse(localStorage.getItem("reminders")) || [];
       var remindersList = document.getElementById("reminders");
       remindersList.innerHTML = "";

       reminders.forEach(function(reminder) {
           var reminderItem = document.createElement("div");
           reminderItem.innerHTML = "<strong>Name:</strong> " + reminder.name + "<br>" +
                                    "<strong>Description:</strong> " + reminder.description + "<br>" +
                                    "<strong>Date:</strong> " + reminder.date + "<br><br>";
           remindersList.appendChild(reminderItem);
       });
   }

This function retrieves all saved reminders from local storage and displays them on the webpage. It clears the existing list of reminders, iterates over each reminder, creates a new HTML element for each reminder, and appends it to the list.

Initialization
**************

.. code-block:: javascript

   // Display reminders when the page loads
   displayReminders();

   document.querySelector('#saveDeadline').addEventListener('click', saveDeadline);

This code block ensures that the list of reminders is displayed when the page loads. It also attaches an event listener to the "Save" button (`#saveDeadline`) so that when clicked, it triggers the `saveDeadline()` function to save a new reminder.

