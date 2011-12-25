Event
=====

Events are Fictures. Fictures are Events. 

**WHAT?**

Specifically, Events associate a collection of photos (a minumum of 6) and 
metadata with a single instance in the time space continum (and the Ficture 
Ecosystem). 

When an Event is created, it's assumed that it will eventually
be published to one of the many Streams on Ficture. Publishing into
streams is handled automatically by our system, and it is dependent on
state attached to the logged in user, such as their privacy settings.

.. http:method:: GET events/{id}/

   :arg id: The ID of the Event to retrieve.

   Returns a single Event.


.. http:method:: POST events/

   :optparam string name: Events can be named with a string up to 55 characters
   :optparam double captured: UTC timestamp (may include sub-second
       intervals)
   :optparam double latitude: Latitude at which Event was captured.
   :optparam double longitude: Longitude at which Event was captured.
   :optparam string ut: The upload token, string of up to 64 characters.
       tokens.
   :optparam integer publish: 0 or 1, whether or not to publish (default=1)
       documentation below on publishing.
   :param file photo-{num}: JPEG photo files
   :param file photo-{num}-meta: JSON photo metadata 

   This method is used to create a new Event in the system. This method
   **ONLY** accepts ``multipart/form-data`` encoded POST body.

   Only logged in users may create Events. Events are associated with the
   logged in user and will by default publish immediately to their
   Streams.

   Upon submitting new Events, our system must do some processing which is
   completed asynchronously, which has the side effect that the entire
   Event is not available immediatly upon return of this method. Instead,
   clients are returned enough data to be able to reference the Event
   until it's finished processing. 
   
   Clients may poll twice a second to check for completed processing.
   Until our system is done processing the Event, a simple data structure
   will be returned::
     
     {'items': [{
        'id': '{EVENT_ID}',
        'type': 'event',
        'status': 'received'}],
      'meta': {}}
   
   That is the most basic form of Event structure which only reports it's
   ID. As the Event passes through our system, the ``status`` field will
   be updated in real time. Typically Events are processed in under 2
   seconds.

   Uploading Files
     Events contain multiple JPEG files, optionally paired with JSON
     metadata. Clients must adhere to the following protocol for Event
     uploads to be interpreted properly.

     * Each file and it's metadata get it's own multipart key
     * Each key and filename must reference the same file.
     * Keys and filenames are referenced by a zero-indexed number. E.g. if
       there are 3 photos in the event, indexes will be ``0, 1, 2``
     * Photos, the actual files containing image data must be keyed
       with **photo-{NUM}** where {NUM} is the zero-indexed index of
       the photo in the set. The filename **MUST** be in the form of
       **photo{NUM}.jpg**
     * Photos **MUST** be ``640x640`` in size. And **MUST** be a JPEG
     * Metadata may be included in the form of a JSON-encoded file with a
       top level dictionary object. If uploading from a camera, this may
       be specially-annotated EXIF data. It **MUST** have the multipart
       key **photo-{NUM}-meta** and **MUST** have the filename
       **photo{NUM}.json**
    
     Example ``multipart/form-data`` body::

        POST /api/v1/films/abcd1234/events/
        Content-Type: multipart/form-data; boundary=--asdf1
        Content-Length: 1234
        --asdf1
        Content-Disposition: form-data; name="photo-0"; filename="photo0.jpg"
        Content-Type: image/jpeg
        
        JPEG DATA....;
        --asdf1
        Content-Disposition: form-data; name="photo-0-meta"; filename="photo0.json"
        Content-Type: application/json
        
        {'some': 'metadata': ['here', 'for photo 2']}
        --asdf1
        Content-Disposition: form-data; name="photo-1"; filename="photo1.jpg"
        Content-Length: 1234

        JPEG DATA....;
        --asdf1
        Content-Disposition: form-data; name="photo-1-meta"; filename="photo1.json"
        Content-Type: application/json

        {'some': 'metadata': ['here', 'for photo 2']}

   Publishing Events
     By default Events are "published," that is, they are inserted into
     the logged in user's relevant streams as the last part of Event
     processing. This behavior may be alterted however by passing ``0``
     for the ``publish`` parameter when creating the Event. In this case,
     our system will process the event fully, but wait until a follow up
     ``PUT`` request is made where ``publish`` is set to ``1`` to insert
     it into the user's streams.

     This is used by the iPhone client to buffer uploads as a user
     captures them, and immediatly upon approving them, publishes them for
     their friends and/or everyone to see.

   Upload Token
     The ``ut`` parameter may be used by your system to signify a single
     upload attempt. Multiple attempts ``POST`` to this method that have
     the same upload token will be denied with a ``409 DUPLICATE ENTRY``

     It's recommended that you ALWAYS include an upload token unless you
     are 100% sure that your requests will NEVER be retried. Event
     creation privliges may be revoked if your application creates
     duplicate Events in the system frequently.


.. http:method:: PUT events/{id}/

   :arg id: The ID of the Event to update.
   :optparam string name: Events can be named with a string up to 55 characters
   :optparam double captured: UTC timestamp (may include sub-second
       intervals)
   :optparam double latitude: Latitude at which Event was captured.
   :optparam double longitude: Longitude at which Event was captured.

   Update metadata about an Event. After an Event is uploaded, certain
   metadata may be updated. Photos attached to an Event however can not be
   updated.

.. http:method:: DELETE events/{id}/

   :arg id: The ID of the Event to delete.

   Removes an Event from Ficture and from the user's relevent Streams.
   Once an Event is removed it **CAN NOT** be restored. We remove the
   Event immediatly from our system. However, it may remain cached in
   clients for however said clients choose to cache them.
