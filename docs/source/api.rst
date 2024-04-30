API
===

(From) uuid.mjs
=======================

This module provides a router for generating UUIDs using the `crypto.randomUUID()` function and handling requests to the endpoint

Router
********

The UUID router handles the following endpoint:

- ``GET /api/uuid``: Generates a UUID and sends it as a JSON response.

Script
********

.. code-block:: javascript

    import { randomUUID } from "crypto";
    import { Router } from "express";

    // Create a new router
    const router = new Router();

    // Route for handling GET requests to '/api/uuid'
    router.get("/", (_req, res) => {
        // Generate a random UUID
        const uuid = randomUUID();
        // Send the UUID as JSON response
        res.send(JSON.stringify({ uuid }));
    });

    // Export the router
    export default router;

This module exports a router that handles GET requests to the `/api/uuid` endpoint by generating a random UUID and sending it as a JSON response.
