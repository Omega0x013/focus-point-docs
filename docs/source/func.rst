Functions and Classes
---------

*(For the web componenents where the constructor has been initialized, this is used to make sure that when you try to attach a html to the shadow DOM it doesnt cause an error if its already defined)*

tagcomponent.mjs
==================

This class represents a custom HTML element called `tag-input`. It provides functionality to add, remove, and store tags.

To use this class, you need to define a custom HTML element with the name `tag-input`. For example:

.. code-block:: html

    <tag-input></tag-input>

Once defined, you can interact with the `tag-input` element using JavaScript.

Class: TagComponent
**********************

:class:`TagComponent` is a custom HTML element that represents a tag input component

Methods
*********

``constructor()``
    Initializes a new instance of the TagComponent class. Sets up initial properties

    .. code-block:: javascript

        constructor() {
            super();
            this.tags = [];
            this.initilized = false;
        }

``prepareHandle()``
    Asynchronously prepares the component for handling user input. Loads the necessary template and sets up event listeners

    .. code-block:: javascript

        async prepareHandle() {
            if (this.initilized) {
                return;
            }
            if (this.shadow === undefined) {
                this.shadow = this.attachShadow({ mode: 'open' });
            }
            const res = await fetch(import.meta.resolve("./tagTemplate.html"));
            this.shadow.innerHTML = await res.text();
            this.shadow.append(this.shadow.querySelector('template').content.cloneNode(true));
            //setting up event listeners    
            this.itemList = this.shadow.querySelector("#itemList");
            this.inputField = this.shadow.querySelector("#newTagInput");
            this.inputField.addEventListener("keyup", this.handleTagAdd.bind(this));
            this.initilized = true;
        }

``connectedCallback()``
    Called when the element is inserted into the DOM, Calls `prepareHandle()` to ensure the component is initialized properly

    .. code-block:: javascript

        async connectedCallback() {
            //setting up DOM
            if (this.initilized == true) {
                return;
            }
            await this.prepareHandle();
        }

``handleTagAdd(event)``
    Handles the addition of a new tag when the user presses the Enter key. Adds the tag to the list of tags and updates the DOM.

    .. code-block:: javascript

        handleTagAdd(e) {
            if (e.key == "Enter") {
                this.addTag(this.inputField.value);
                this.itemList.prepend(this.inputField);
                this.inputField.focus();
            }
        }

``getTags()``
    Returns the list of tags currently stored in the component.

    .. code-block:: javascript

        getTags() {
            return this.tags;
        }

``storeTag(tagName, id)``
    Stores a tag in the local storage and asociates the tag with an identifier.

    .. code-block:: javascript

        storeTag(tagName, id) {
            // add it form local storage
            let localTags = JSON.parse(localStorage["tags"]);
            localTags[tagName] == null ? localTags[tagName] = [id] : localTags[tagName].push(id);
            localStorage["tags"] = JSON.stringify(localTags);
        }

``setTags(textList)``
    Sets the initial tags for the component based on the provided list. Updates the DOM accordingly.

    .. code-block:: javascript

        setTags(textList) {
            if (textList == null) {
                return;
            }
            this.itemList = this.shadow.querySelector("#itemList");
            for (let tag of textList) {
                this.addTag(tag);
            }
            this.itemList.prepend(this.inputField);
        }

``removeTag(tagObj)``
    Removes a tag from the component and updates the DOM.

    .. code-block:: javascript

        removeTag(tagObj) {
            this.tags = this.tags.filter(
                (tags) => tags !== tagObj.querySelector('p').textContent,
            );
            tagObj.remove();
        }

``addTag(inputText)``
    Adds a new tag to the component. Updates the DOM and emits a `tagAdded` event.

    .. code-block:: javascript

        addTag(inputText) {
            const tag = document.createElement('li');
            const text = document.createElement('p');
            const remove = document.createElement('button');
            text.textContent = inputText;
            remove.textContent = 'X';
            tag.append(text, remove);
            this.itemList.prepend(tag);
            this.tags.push(inputText);
            this.inputField.value = '';

Events
*******

``tagAdded``
    Emitted when a new tag is added to the component. Contains the details of the added tag.

``tagRemoved``
    Emitted when a tag is removed from the component. Contains the details of the removed tag.

timercomponent.mjs
==================================

This class represents a custom HTML element called `timer-component`. It provides functionality for a study timer with options to set study and rest times.

To use this class, you need to define a custom HTML element with the name `timer-component`. For example:

.. code-block:: html

    <timer-component></timer-component>

Once defined, you can interact with the `timer-component` element using JavaScript.

Class: TimerComponent
***********************

:class:`TimerComponent` is a custom HTML element that represents a study timer component.

Methods
********

``constructor()``
    Initializes a new instance of the TimerComponent class.

    .. code-block:: javascript

        constructor() {
            super();
        }

``connectedCallback()``
    Called when the element is inserted into the DOM. Sets up the timer component by fetching the template, initializing variables, and adding event listeners.

    .. code-block:: javascript

        async connectedCallback() {
            // Everything from the constructor
            this.shadow = this.attachShadow({ mode: 'open' });

            // Fetch the template file, and add it to the shadow dom
            const res = await fetch(import.meta.resolve("./TimerComponent.html"));
            this.shadow.innerHTML = await res.text();

            // Construct the actual element
            this.shadow.append(this.shadow.querySelector('template').content.cloneNode(true));

            // Initialize variables and set up event listeners
            this.titleText = this.shadow.querySelector("#title");
            this.clock = this.shadow.querySelector("#clock");
            this.time = this.shadow.querySelector("#studyTime");
            this.restTime = this.shadow.querySelector("#restTime");
            this.start = this.shadow.querySelector("#start");
            this.stop = this.shadow.querySelector("#stop");
            this.restInput = this.shadow.querySelector("#restTimeInput");
            this.restLabel = this.shadow.querySelector("#restTimeLabel");
            this.studyInput = this.shadow.querySelector("#studyTimeInput");
            this.settings = this.shadow.querySelector("#settings");
            this.close = this.shadow.querySelector("#close");
            this.clockContainer = this.shadow.querySelector("#clockContainer");
            this.radius = 100;
            this.circumference = 2 * this.radius * Math.PI;
            this.seconds = 0;
            this.timerList = [];
            this.isTimerRunning = false;

            // Set up initial states and event listeners
            this.titleText.textContent = "study timer";
            this.stop.classList.add("hidden");
            this.start.addEventListener("click", this.startTimer.bind(this));
            this.studyInput.addEventListener("input", this.updateStudyTime.bind(this));
            this.restInput.addEventListener("input", this.updateRestTime.bind(this));
            this.stop.addEventListener("click", this.stopTimer.bind(this));
            this.close.addEventListener("click", this.destroySelf.bind(this));
            this.classList.add("popup-active");
            this.clock.style.strokeDasharray = this.circumference;
            this.updateRestTime();
            this.clockContainer.classList.add("hidden");
            this.updateStudyTime();
        }

``destroySelf()``
    Removes the timer component from the DOM.

    .. code-block:: javascript

        destroySelf() {
            this.remove();
        }

``updateStudyTime()``
    Updates the displayed study time based on the input value

    .. code-block:: javascript

        updateStudyTime() {
            const formattedTime = this.getFormattedTimeFromSeconds(this.studyInput.value * 60);
            this.time.textContent = `${formattedTime} study time`;
        }

``updateRestTime()``
    Updates the displayed rest time based on the input value

    .. code-block:: javascript

        updateRestTime() {
            const formattedTime = this.getFormattedTimeFromSeconds(this.restInput.value * 60);
            this.restTime.textContent = `${formattedTime} rest time`;
        }

``getFormattedTimeFromSeconds(seconds)``
    Converts seconds into a formatted time string (HH:MM:SS).

    .. code-block:: javascript

        getFormattedTimeFromSeconds(seconds) {
            return new Date(seconds * 1000).toISOString().slice(11, 19);
        }

``stopTimer()``
    Stops the timer and resets the component to its initial state.

    .. code-block:: javascript

        stopTimer() {
            this.settings.classList.remove("hidden");
            this.stop.classList.add("hidden");
            this.restTime.classList.remove("hidden");
            this.start.textContent = "start";
            clearInterval(this.intervalID);
            this.seconds = 0;
            this.updateTimerDisplay();
            this.isTimerRunning = false;
            this.timerList = [];
            const studyFormattedTime = this.getFormattedTimeFromSeconds(this.studyInput.value * 60);
            const restFormattedTime = this.getFormattedTimeFromSeconds(this.restInput.value * 60);
            this.time.textContent = `${studyFormattedTime} study time`;
            this.restTime.textContent = `${restFormattedTime} rest time`;
            this.titleText.textContent = "study timer";
            this.close.classList.remove("hidden");
            this.clockContainer.classList.add("hidden");
        }

``incrementTimer()``
    Increments the timer by one second and updates the display, Handles switching between study and rest periods

    .. code-block:: javascript

        incrementTimer() {
            if (this.getFormattedTimeFromSeconds(this.timerList[0]) == this.getFormattedTimeFromSeconds(this.seconds)) {
                if (this.timerList.length > 1) {
                    // Switch over to break timer
                    this.titleText.textContent = "break";
                    this.seconds = 0;
                    this.timerList.shift();
                    this.clock.style.stroke = "red";
                } else {
                    // End the timer
                    this.stopTimer();
                    return;
                }
            }
            this.seconds++;
            this.updateTimerDisplay();
        }

``startTimer()``
    Starts or pauses the timer based on its current state.

    .. code-block:: javascript

        startTimer() {
            if (!this.isTimerRunning) {
                if (this.timerList[0] == undefined) {
                    this.clockContainer.classList.remove("hidden");
                    // On the first time you click start
                    this.timerList.push(this.studyInput.value * 60);
                    this.timerList.push(this.restInput.value * 60);
                    this.titleText.textContent = "study";
                    this.close.classList.add("hidden");
                    this.clock.style.stroke = "blue";
                }
                this.intervalID = setInterval(this.incrementTimer.bind(this), 1000);
                this.isTimerRunning = true;
                this.start.textContent = "pause";
                this.settings.classList.add("hidden");
                this.stop.classList.remove("hidden");
            } else {
                this.start.textContent = "start";
                clearInterval(this.intervalID);
                this.isTimerRunning = false;
            }
        }

``updateTimerDisplay()``
    Updates the timer display with the current time and progress

    .. code-block:: javascript

        updateTimerDisplay() {
            const formattedTime = this.getFormattedTimeFromSeconds(this.seconds);
            this.time.innerText = `${formattedTime} / ${this.getFormattedTimeFromSeconds(this.timerList[0])}`;
            const percent = (this.seconds / this.timerList[0]) * 100;
            this.setProgress(percent);
        }

``setProgress(percent)``
    Sets the progress of the timer based on the provided percentage.

    .. code-block:: javascript

        setProgress(percent) {
            this.clock.style.strokeDashoffset = this.circumference - (percent / 100) * this.circumference;
        }

``disconnectedCallback()``
    Called when the element is removed from the DOM. Clears the interval to stop the timer.

    .. code-block:: javascript

        disconnectedCallback() {
            clearInterval(this.intervalID);
        }

timetableevent.mjs
==================================

This class represents a custom HTML element called `timetable-event`. It provides functionality for displaying and interacting with timetable events.

To use this class, you need to define a custom HTML element with the name `timetable-event`. For example:

.. code-block:: html

    <timetable-event 
        description="Event Description" 
        eventTitle="Event Title" 
        eventID="1" 
        startDate="2024-04-30T09:00:00" 
        endDate="2024-04-30T10:00:00" 
        timetable="example-timetable" 
        tag="tag1,tag2">
    </timetable-event>

Once defined, you can interact with the `timetable-event` element using javascript

Class: TimetableEvent
***********************

:class:`TimetableEvent` is a custom HTML element that represents a timetabled event

Methods
*********

``constructor()``
    Initializes a new instance of the TimetableEvent class. Sets up the shadow DOM and initializes properties based on attributes.

    .. code-block:: javascript

        constructor() {
            super();
            this.shadow = this.attachShadow({ mode: 'open' });
            this.shadow.innerHTML = entryTemplate.innerHTML;
            this.LOCALKEY = "timetableEvents";
            this.eventID = this.getAttribute("eventID");
            this.description = this.getAttribute("description");
            this.startDate = this.getAttribute("startDate");
            this.endDate = this.getAttribute("endDate");
            this.eventTitle = this.getAttribute("eventTitle");
            this.tag = this.getAttribute("tag");
        }

``connectedCallback()``
    Called when the element is inserted into the DOM. Adds a click event listener to open the event options pop-up

    .. code-block:: javascript

        async connectedCallback() {
            this.addEventListener("click", async () => { await this.eventOptionsPopup() });
        }

``eventOptionsPopup()``
    Creates a popup for editing event options.

    .. code-block:: javascript

        async eventOptionsPopup() {
            const template = await util.fetchTemplate("./addEventTemplate.html");
            let timetableHolder;
            timetableHolder = document.querySelector("body");
            template.classList.add("popup-active");
            timetableHolder.prepend(template);
            let h1Title = document.querySelector("#popupTitle");
            let title = document.querySelector("#eventTitleInput");
            let description = document.querySelector("#EventdescriptionInput");
            const deleteButton = document.createElement('button');
            let editButton = document.querySelector("#selfSave");

            this.tagInput = document.createElement('tag-input');
            await this.tagInput.prepareHandle();
            editButton.parentNode.insertBefore(this.tagInput, editButton);
            this.tag = util.getTagsFromLocal(this.eventID);
            this.tagInput.setTags(this.tag);
            title.value = this.eventTitle;

            description.value = this.description;
            deleteButton.textContent = "delete";
            h1Title.textContent = "edit event";
            h1Title.parentNode.appendChild(deleteButton);
            editButton.textContent = "confirm edit";
            util.fillEntryPopup(new Date(this.startDate), new Date(this.endDate));
            deleteButton.addEventListener("click", this.deleteThisEvent.bind(this));
            editButton.addEventListener("click", this.editThisEvent.bind(this));
        }

``editThisEvent()``
    Handles editing the event and updates event details and refreshes the timetable display.

    .. code-block:: javascript

        async editThisEvent() {
            this.startDate = new Date(document.querySelector("#startDateInput").value);
            this.eventTitle = document.querySelector("#eventTitleInput").value;
            this.description = document.querySelector("#EventdescriptionInput").value;
            this.endDate = new Date(document.querySelector("#endDateInput").value);
            util.storeEvent(this);
            util.clearEvents();
            util.cleanTags(this.eventID);
            for (let tag of this.tagInput.getTags()) {
                this.tagInput.storeTag(tag, this.eventID);
            }
            util.populateTableFromLocal();
            util.selfDelete();
        }

``deleteThisEvent()``
    Deletes the event and refreshes the timetable display

    .. code-block:: javascript

        deleteThisEvent() {
            util.clearEvents();
            this.remove();
            util.deleteEvent(this);
            util.selfDelete();
            util.cleanTags(this.eventID);
            util.populateTableFromLocal();
        }

``setTitle()``
    Sets the title of the event.

    .. code-block:: javascript

        setTitle() {
            this.shadowRoot.append(`${this.eventTitle}`);
        }

timetable_template.mjs
============================

The `entryTemplate` defines the HTML template for a timetable event entry. This documentation provides an overview of its structure.

Template Definition
*********************

.. code-block:: javascript

   export const entryTemplate = document.createElement("template");
   entryTemplate.innerHTML = `
   <style>


   </style>
   <div class="entry">
   </div>
   `

This template defines the structure of a timetable event entry

Structure
**********

The template consists of the following elements:

- `<style>`: This element contains CSS styles specific to the timetable entry. (Currently empty)
- `<div class="entry">`: This element serves as the container for the timetable entry content.

timetablesUtils.mjs
=====================================

The `timetableEvents` utility module provides various functions for managing timetable events, such as saving, deleting, and displaying events. This documentation provides an overview of its functionalities and usage.

Constants
***********

- `LOCALKEY`: The key used to store timetable events in the local storage.
- `HOUROFFSET`: The hour offset used for calculating grid rows in the timetable display.
- `DURATIONTHRESHOLD`: The threshold for detecting a swipe duration.
- `MOVETHRESHOLD`: The threshold for detecting swipe movements.

Functions
**********

``getNextDays(date, days)``
   
   Generates an array of dates for the next specified number of days starting from a given date
   
.. code-block:: javascript
   
      export function getNextDays(date, days) {
        let array = [];
        for (let i = 0; i < days; i++) {
          let tempDate = new Date(date);
          tempDate.setHours(0);
          tempDate.setMinutes(0);
          tempDate.setSeconds(0);
          tempDate.setDate(date.getDate() + i);
          array.push(tempDate);
        }
        return array
      }

``saveDates()``
   
   Saves event data entered in the add new event popup to the local storage
   
.. code-block:: javascript

    export function saveDates() {
        // Save dates from the add new event popup
        let entry = document.createElement("timetable-event");
        entry.eventID = crypto.randomUUID();
        entry.eventTitle = document.querySelector("#eventTitleInput").value;
        entry.description = document.querySelector("#EventdescriptionInput").value;
        entry.startDate = new Date(document.querySelector("#startDateInput").value);
        entry.endDate = new Date(document.querySelector("#endDateInput").value);
        const tagInput = document.querySelector("tag-input");
        entry.tag = tagInput.getTags();
        storeEvent(entry);
        for (let tag of entry.tag) {
            tagInput.storeTag(tag, entry.eventID);
        }
        clearEvents();
        selfDelete();
        populateTableFromLocal();
    }

``selfDelete()``
   
   Removes the active popup from the DOM
   
.. code-block:: javascript

    export function selfDelete() {
        let popup = document.querySelector(".popup-active");
        popup.remove();
    }


``deleteEvent(event)``
   
   Deletes a specific event from the local storage.
   
.. code-block:: javascript

    export function deleteEvent(event) {
        const LOCALKEY = "timetableEvents";
        let tempStore = { "events": [] };
        if (localStorage[LOCALKEY] != null) {
            tempStore = JSON.parse(localStorage[LOCALKEY]);
        }
        let tempStoreEvents = tempStore["events"].map(item => item.eventID);
        if (tempStoreEvents.includes(event.eventID)) {
            // Replaces old events with newer version of event ID
            let index = tempStoreEvents.indexOf(event.eventID);
            tempStore["events"].pop(index);
            localStorage[LOCALKEY] = JSON.stringify(tempStore);
        } else {
            // Event doesn't exist
        }
    }

``storeEvent(event)``
   
   Stores a timetable event in the local storage
   
.. code-block:: javascript

    export function storeEvent(event) {
        const LOCALKEY = "timetableEvents";
        let tempStore = { "events": [] };
        if (localStorage[LOCALKEY] != null) {
            tempStore = JSON.parse(localStorage[LOCALKEY]);
        }
        let tempStoreEvents = tempStore["events"].map(item => item.eventID);
        if (tempStoreEvents.includes(event.eventID)) {
            // Replaces old events with newer version of event ID
            let index = tempStoreEvents.indexOf(event.eventID);
            tempStore["events"][index] = event;
        } else {
            // Makes sure eventID is unique
            tempStore["events"].push(event);
        }
        localStorage[LOCALKEY] = JSON.stringify(tempStore);
    }


``fillEntryPopup(startDate, endDate)``
   
   Fills the add new event popup with default or specified start and end dates.
   
.. code-block:: javascript

    export function fillEntryPopup(startDate, endDate) {
        let startDateInput = document.querySelector("#startDateInput");
        let endDateInput = document.querySelector("#endDateInput");
        startDateInput.value = convertDateToInputValue(startDate);
        endDateInput.value = convertDateToInputValue(endDate);

        let closeButton = document.querySelector("#selfDelete");
        closeButton.addEventListener("click", selfDelete);
    }

``convertDateToInputValue(date)``
   
   Converts a Javascript date object to a string format suitable for input fields
   
.. code-block:: javascript

    export function convertDateToInputValue(date) {
        let minutes = `${date.getMinutes()}`.padStart(2, "0");
        let days = `${date.getDate()}`.padStart(2, "0");
        let hour = `${date.getHours()}`.padStart(2, "0");
        let month = `${date.getMonth() + 1}`.padStart(2, "0");
        let currentDate = `${date.getFullYear()}-${month}-${days}T${hour}:${minutes}`;
        return currentDate;
    }


``fetchTemplate(path)``
   
   Fetches an HTML template from the specified path and returns it as a DOM element
   
.. code-block:: javascript

    export async function fetchTemplate(path) {
        const res = await fetch(import.meta.resolve(path));
        let template = document.createElement("div");
        template.innerHTML = await res.text();
        template.append(template.querySelector("template").content.cloneNode(true));
        return template;
    }

``displayPreviousDays()``
   
    Displays the previous set of days on the timetable based on the currently selected vieww
   
.. code-block:: javascript

    function displayPreviousDays() {
        let selected = document.querySelector('input[name="timetable"]:checked');
        let parentNode = selected.parentNode;
        let timetableLength = parseInt(parentNode.dataset.daylength);
        clearEvents();
        let date = new Date(sessionStorage.getItem("currentDate"));
        date.setDate(date.getDate() - timetableLength);
        sessionStorage.setItem("currentDate", date);
        setCurrentMonth(date);
        fillTimetableDays(date);
        populateTableFromLocal();
    }


``displayNextDays()``
   
    Displays the next set of days on the timetable based on the currently selected view
   
.. code-block:: javascript

    function displayNextDays() {
        let selected = document.querySelector('input[name="timetable"]:checked');
        let parentNode = selected.parentNode;
        let timetableLength = parseInt(parentNode.dataset.daylength);
        clearEvents();
        let date = new Date(sessionStorage.getItem("currentDate"));
        increaseDate(date, (timetableLength) * 24);
        sessionStorage.setItem("currentDate", date);
        setCurrentMonth(date);
        fillTimetableDays(date);
        populateTableFromLocal();
    }

``timetableNavigation(nextSelector, previousSelector)``
   
   Sets up navigation functionality for switching between timetable view
   
.. code-block:: javascript

    export function timetableNavigation(nextSelector, previousSelector) {
        let next = document.querySelector(nextSelector);
        let previous = document.querySelector(previousSelector);
        let timetables = document.querySelectorAll('.timetable-wrapper');
        let firstTouchTime, posX, posY;

        function startTouch(event) {
            firstTouchTime = Date.now();

            // Record the X and Y coordinates of the touch on the screen
            posY = event.touches[0].clientY;
            posX = event.touches[0].clientX;
        }

        function endTouch(event) {
            const touchEndTime = Date.now();
            swipe(event, touchEndTime - firstTouchTime);
        }

        function swipe(event, duration) {
            // Get the final X and Y coordinates of the touch
            const endClientX = event.changedTouches[0].clientX;
            const endClientY = event.changedTouches[0].clientY;
            if (duration <= DURATIONTHRESHOLD) {
                if (posY - endClientY >= MOVETHRESHOLD) {
                    // Swipe up detected
                    console.log('swiped up');
                } else if (endClientY - posY >= MOVETHRESHOLD) {
                    console.log('swiped down');
                } else if (endClientX - posX >= MOVETHRESHOLD) {
                    console.log('swiped right');
                    displayPreviousDays();
                } else if (posX - endClientX >= MOVETHRESHOLD) {
                    console.log('swiped left');
                    displayNextDays();
                }
            }
        }

        for (let timetable of timetables) {
            timetable.addEventListener("touchstart", startTouch);
            timetable.addEventListener("touchend", endTouch);
        }

        next.addEventListener("click", displayNextDays);
        previous.addEventListener("click", displayPreviousDays);
    }


``fillTimetableDays(currentDate)``
   
    Fills the timetable display with day headings based on the current date
   
.. code-block:: javascript

    export function fillTimetableDays(currentDate) {
        let timetables = document.querySelectorAll(".timetable-wrapper");
        for (let timetable of timetables) {
            let numOfColumns = timetable.dataset.daylength;
            let days = timetable.querySelectorAll(".day-heading");
            let weekStart = new Date(currentDate);
            if (numOfColumns == 3) {
                // Handle for 3 columns
            } else if (numOfColumns == 7) {
                weekStart = new Date(currentDate);
                if (currentDate.getDay() != 1) {
                    let diff = Math.abs(1 - currentDate.getDay());
                    weekStart.setDate(currentDate.getDate() - (currentDate.getDay()));
                }
            }
            let dates = getNextDays(weekStart, numOfColumns);
            const dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
            for (let i = 0; i < numOfColumns; i++) {
                days[i].dataset.day = dates[i];
                days[i].textContent = `${dayNames[dates[i].getDay()].substring(0, 3)}   ${dates[i].getDate()}`;
            }
        }
    }

``setCurrentMonth(date)``
   
    Sets the display to show the current month and year based on the given date.
   
.. code-block:: javascript

    export function setCurrentMonth(date) {
        const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];

        let monthCounter = document.querySelector("#monthDisplay");
        monthCounter.textContent = ` ${months[date.getMonth()]}  ${date.getFullYear()}`;
    }

``clearEvents()``
   
    Clears all timetable events from the display.
   
.. code-block:: javascript

    export function clearEvents() {
        let events = document.querySelectorAll("timetable-event");
        let dayHeading = document.querySelectorAll(".day-heading");
        Array.from(events).map(entry => entry.remove());
        Array.from(dayHeading).map(entry => entry.dataset.objectnumber = 0);
    }



``multipleDaySort(a, b)``
   
    Sorts events based on their duration and start date
   
.. code-block:: javascript

    export function multipleDaySort(a, b) {
        let condition = a.dateDiff >= b.dateDiff && a.startDate <= b.startDate;
        if (condition) {
            return -1;
        } else if (!condition) {
            return 1;
        }
    }



``increaseDate(date, hours)``
   
    Increases the given date by a specified number of hours.
   
.. code-block:: javascript

    export function increaseDate(date, hours) {
        // Increases the date by x number of hours
        date.setHours(date.getHours() + hours);
        return date;
    }



``populateTableFromLocal()``
   
    Populates the timetable display with events stored in the local storage.
   
.. code-block:: javascript

    export function populateTableFromLocal() {
        let timetables = document.querySelectorAll(".timetable-wrapper");
        let localEvents = JSON.parse(localStorage[LOCALKEY]);
        if (localEvents.events.length < 1) {
            return;
        }
        for (let timetable of timetables) {
            let entries = [];
            for (let event of localEvents["events"]) {
                let entry = document.createElement("timetable-event");
                entry.eventID = event.eventID;
                entry.eventTitle = event.eventTitle;
                entry.description = event.description;
                entry.startDate = new Date(event.startDate);
                entry.endDate = new Date(event.endDate);
                entry.dateDiff = getDifferenceDays(entry.startDate, entry.endDate);
                entry.tag = getTagsFromLocal(entry.eventID);
                entry.eventTitle = event.eventTitle;
                entries.push(entry);
            }
            entries.sort(multipleDaySort);
            for (let entry of entries) {
                addToTimetable(timetable, entry);
            }
            let hourHeading = timetable.querySelector("#multi-day-events");
            let biggest = Math.max(...Array.from(timetable.querySelectorAll(".day-heading")).map(column => column.dataset.objectnumber));
            hourHeading.style.paddingTop = `${biggest * 10}px`;
        }
    }



``addEventPopup()``
   
    Displays the add new event popup.
   
.. code-block:: javascript

    export async function addEventPopup() {
        let timetableHolder;
        await fetchTemplate("./addEventTemplate.html").then(res => {
            timetableHolder = document.querySelector("body");
            res.classList.add("popup-active");
            timetableHolder.prepend(res);
            let currentDate = new Date();
            let nextHour = new Date();
            nextHour.setHours(currentDate.getHours() + 1);
            let saveButton = document.querySelector("#selfSave");
            saveButton.addEventListener("click", saveDates);
            fillEntryPopup(currentDate, nextHour);
            const editButton = res.querySelector("#selfSave");
            const tagInput = document.createElement('tag-input');
            editButton.parentNode.insertBefore(tagInput, editButton);
        });
    }


``getMaxDayOnTimetable(timetable)``
   
    Gets the maximum date displayed on the timetable.
   
.. code-block:: javascript

    function getMaxDayOnTimetable(timetable) {
        let maxDay = new Date(timetable.querySelector(".day-heading").dataset.day);
        maxDay = increaseDate(maxDay, parseInt(timetable.dataset.daylength) * 24);
        maxDay.setHours(24);
        return maxDay;
    }



``getMinDayOnTimetable(timetable)``
   
    Gets the minimum date displayed on the timetable
   
.. code-block:: javascript

    function getMinDayOnTimetable(timetable) {
        let minDay = new Date(timetable.querySelector(".day-heading").dataset.day);
        minDay.setHours(0);
        return minDay;
    }


``getDifferenceDays(startDate, endDate)``
   
    Calculates the difference in days between two dates
   
.. code-block:: javascript

    export function getDifferenceDays(startDate, endDate) {
        const milisecondDiff = Math.abs(endDate - startDate);
        let dayDiff = Math.floor(milisecondDiff / (1000 * 60 * 60 * 24));
        let sameDay = endDate.getDate() == startDate.getDate() && endDate.getMonth() == startDate.getMonth() && endDate.getFullYear() == startDate.getFullYear();
        if (dayDiff < 1 && sameDay) {
            // It's one day
            dayDiff = 0;
        } else if (dayDiff < 1) {
            dayDiff = 1;
        }
        return dayDiff;
    }



``addToTimetable(timetable, entry)``
   
    Adds a timetable event to the display grid.
   
.. code-block:: javascript

    export function addToTimetable(timetable, entry) {
        let maxDay = getMaxDayOnTimetable(timetable);
        let minDay = getMinDayOnTimetable(timetable);
        let endDate = new Date(entry.endDate);
        let startDate = new Date(entry.startDate);
        if (!((startDate <= maxDay) && (minDay <= endDate))) {
            // Event is not in range on the timetable then don't add it
            return;
        }
        // Event is definitely in the current dates on the timetable
        let dateDiff = getDifferenceDays(startDate, endDate);
        let days = timetable.querySelectorAll(".day-heading");
        entry.classList.add("entry");
        for (let day of days) {
            let dayNum = new Date(day.dataset.day);
            if (minDay >= startDate && dayNum.getDate() == minDay.getDate() || dayNum.getDate() == startDate.getDate()) {
                let columnIndex = parseInt(day.dataset.columnindex);
                if (dateDiff > 0) {
                    // If the event spans over multiple days
                    entry.classList.add("multi-day");
                    let startColumn = columnIndex;
                    if (minDay >= startDate) {
                        // Start date is before the first date on the timetable set it to the first date on the timetable
                        startColumn = parseInt(timetable.querySelector(".day-heading").dataset.columnindex);
                        dateDiff = dateDiff - getDifferenceDays(startDate, minDay) - 1;
                    }
                    let endColumn = columnIndex + dateDiff + 1;
                    let maxColumn = parseInt(timetable.querySelector(".day-heading").dataset.columnindex) + parseInt(timetable.dataset.daylength);
                    if (endColumn >= maxColumn) {
                        // Add class here making it show that it extends over these days
                        endColumn = maxColumn;
                    }
                    entry.style = `grid-column: ${startColumn} / ${endColumn} ; grid-row: ${entry.HOUROFFSET - 1} / ${entry.HOUROFFSET - 1} ;`;
                    entry.style.marginTop = `${parseInt(day.dataset.objectnumber) * 15}px`;
                    for (let i = new Date(dayNum); i < increaseDate(new Date(dayNum), (parseInt(timetable.dataset.daylength) - 1) * 24); i = increaseDate(i, 24)) {
                        // Set the hours, minutes, and seconds of each day to 0 so we can correctly select it 
                        i.setHours(0);
                        i.setMinutes(0);
                        i.setSeconds(0);
                        let header = timetable.querySelector(`[data-day="${i}"]`);
                        if (header != null) {
                            // For each column header add 1 to it for each event it has
                            // Might need to reset it 
                            header.dataset.objectnumber = parseInt(header.dataset.objectnumber) + 1;
                        }
                    }
                } else {
                    // It overlaps on the hours offset with the same method  
                    entry.style = `grid-column: ${columnIndex} / ${columnIndex} ; grid-row: ${startDate.getHours() + HOUROFFSET} / ${endDate.getHours() + HOUROFFSET} ;`;
                }
                day.parentNode.append(entry);
                storeEvent(entry);
                break;
            }
        }
    }



``getTagsFromLocal(id)``
   
    Retrieves tags associated with a specific event ID from local storage.
   
.. code-block:: javascript

    export function getTagsFromLocal(id) {
        let localTags = JSON.parse(localStorage["tags"]);
        const tags = [];
        for (let tagName of Object.keys(localTags)) {
            if (localTags[tagName].includes(id)) {
                tags.push(tagName);
            }
        }
        return tags;
    }



``cleanTags(id)``
   
    Removes tags associated with a specific event id from local storage
   
.. code-block:: javascript

    export function cleanTags(id) {
        const cachedTags = JSON.parse(localStorage["tags"]);
        for (let tag of Object.keys(cachedTags)) {
            cachedTags[tag] = cachedTags[tag].filter((item) => item !== id);
        }
        localStorage["tags"] = JSON.stringify(cachedTags);
    }


``init()``
   
    initializes the timetable display and sets up event listeners for navigation and adding new events
   
.. code-block:: javascript

    export function init() {
        sessionStorage.setItem("currentDate", new Date());
        let tempStore = { "events": [] };
        // let tempStore = { };
        if (localStorage[LOCALKEY] != null) {
            tempStore = JSON.parse(localStorage[LOCALKEY]);
        }
        localStorage[LOCALKEY] = JSON.stringify(tempStore);
        let tagStore = {};
        if (localStorage["tags"] != null) {
            tagStore = JSON.parse(localStorage["tags"]);
        }
        localStorage["tags"] = JSON.stringify(tagStore);

        let date = new Date();
        fillTimetableDays(date);
        setCurrentMonth(date);
        sessionStorage.setItem("currentDate", new Date());

        timetableNavigation("#nextTimetable", "#previousTimetable");
        populateTableFromLocal();
        let addEvent = document.querySelector("#addEvent");
        addEvent.addEventListener("click", async () => {
            addEventPopup();
        });
    }


main.mjs
====================

This script initializes the timetable utilities and handles the creation of a timer component when a button is clicked.

Script
*******

.. code-block:: javascript

    import * as util from "./components/TimetableEvent/timetablesUtils.mjs";

    // Initialize timetable utilities
    util.init();

    // Get the button element
    const timerButton = document.querySelector("#timerButton");

    // Add event listener to the button
    timerButton.addEventListener("click", () => {
        // Create a timer component
        const timerlater = document.createElement("timer-component");
        
        // Get the navigation bar element
        const nav = document.querySelector(".nav-bar");
        
        // Append the timer component to the navigation bar
        nav.appendChild(timerlater);
    });

This script initializes the timetable utilities module and listens for clicks on the button with the ID ``timerButton``. When clicked, it creates a new instance of the timer component and appends it to the navigation bar

serviceworker.js
============================

This service worker script listens for the installation and activation events and logs messages when they occur.

Script
********

.. code-block:: javascript

    self.addEventListener("install", () => console.log("Installed"));
    self.addEventListener("activate", () => console.log("Activated"));

This service worker script listens for the installation and activation events. When the service worker is installed, it logs the message "Installed", and when it is activated, it logs the message "Activated".

index.mjs
=======================

This module sets up an Express application with static file serving and API routing for generating UUIDs.

Module
*********

The ``index.mjs`` module sets up an Express application with the following features:

1. Serving static files from the ``public`` directory.
2. Handling API requests for generating UUIDs.

Script
*******

.. code-block:: javascript

    import express from "express";
    import uuid from "./routes/uuid.mjs";

    // Create an Express application
    const app = express();

    // Set the port
    const PORT = process.env.PORT || 8080;

    // Serve static files from the 'public' directory
    app.use(express.static(`${import.meta.dirname}/public`));

    // Route for generating UUIDs
    app.use("/api/uuid/", uuid);

    // Start the Express server
    if (import.meta.url.endsWith(process.argv[1])) {
        app.listen(PORT, () => {
            console.log(`Listening on port ${PORT}`);
        });
    }

This module sets up an Express application, defines routes for serving static files and handling API requests, and starts the server if the script is run directly.
