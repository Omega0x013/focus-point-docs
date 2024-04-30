Functions and Classes
---------

tagcomponent.mjs
==================

This class represents a custom HTML element called `tag-input`. It provides functionality to add, remove, and store tags.

Usage
------

To use this class, you need to define a custom HTML element with the name `tag-input`. For example:

.. code-block:: html

    <tag-input></tag-input>

Once defined, you can interact with the `tag-input` element using JavaScript.

Class: TagComponent
--------------------

:class:`TagComponent` is a custom HTML element that represents a tag input component.

Methods
~~~~~~~

``constructor()``
    Initializes a new instance of the TagComponent class. Sets up initial properties.

    .. code-block:: javascript

        constructor() {
            super();
            this.tags = [];
            this.initilized = false;
        }

``prepareHandle()``
    Asynchronously prepares the component for handling user input. Loads the necessary template and sets up event listeners.

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
    Called when the element is inserted into the DOM. Calls `prepareHandle()` to ensure the component is initialized properly.

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
    Stores a tag in the local storage. Associates the tag with an identifier.

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
            remove.addEventListener('click', () => {
                this.removeTag(tag);
            });
            // Emit a tagAdded event
            this.dispatchEvent(new CustomEvent('tagAdded', { detail: { tag: inputText } }));
        }

Events
~~~~~~

``tagAdded``
    Emitted when a new tag is added to the component. Contains the details of the added tag.

``tagRemoved``
    Emitted when a tag is removed from the component. Contains the details of the removed tag.


