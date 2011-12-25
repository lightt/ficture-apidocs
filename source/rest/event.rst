Events
======

Events are Fictures. Fictures are Events. 

**WHAT?**

Specifically, Events associate a collection of photos (a minumum of 6) and 
metadata with a single instance in the timespace continuum (and the Ficture 
Ecosystem). 

When an Event is created, it's assumed that it will eventually
be published to one of the many Streams on Ficture. Publishing into
streams is handled automatically by our system, and it is dependent on
state attached to the logged in user, such as their privacy settings.

.. http:response:: Event object

   :data string id: The unique identifier for this event. Typically 22-23
       characters
   :data integer seen: Whether or not this has been seen by the logged in
       user - user must be logged in for this to be present.
   :data string username: The fully rendered name of the user who
       captured this event.
   :data string location_name: The fully rendered name of the location
       where the user captured this event. May be empty.
   :data integer user: ID of user who captured the event.
   :data double created: Double-precision floating point UTC timestamp of
       when the Event was captured.
   :data integer privacy: The privacy of this Event. See Privacy.
   :data double latitude: Latitude where this Event was captured
   :data double longitude: Longitude where this Event was captured
   :data integer number_of_frames: Total frames in this Event
   :data string timesince: Fully rendered "time since" string that
       describes the time that has passed since the Event was captured.
   :data hash strips: Strips hash, see **Strips** below
   :data hash batches: Batches hash, see **Batches** below
   :data array frames: Frames array, see **Frames** below
   :format: JSON

   Event objects come in two forms. The most readable and approachable,
   the full representation, is shown below. It includes full URLs, and a
   large amount of metadata intact. Not surprisingly, it's also the most
   expensive to generate, largest to download and slowest to work with.
   Therefore it's recommended to use full Event objects only when all this
   information is actually used. E.g. downloading huge Streams of full
   Events will get your app banned from the API pretty quickly.

   A full Event object::

        {
           "username": "Earl Shuman",
           "user": 322,
           "location_name": "Orland, CA",
           "name": "untitled",
           "privacy": 1,
           "timesince": "2 hours ago",
           "created": 132478167193295,
           "longitude": -122.19111,
           "latitude": 39.73517,
           "seen": 1,
           "number_of_frames": 13,
           "id": "DFHCjcLqMR4adGEjE7CMHh",
           "status": "published",
           "strips": {
             "{{SIZE}}": { # one hash per size
               "urls": ["{{FRAME_BASEURL}}/{{EVENT_ID}}/{{SIZE}}1-13.jpg"],
               "indexes": [[1, 13]]
             }
           },
           "batches": {
             "{{SIZE}}": { # one hash per size
               "urls": ["{{FRAME_BASEURL}}/{{EVENT_ID}}/{{SIZE}}1-13.bjpeg"],
               "indexes": [[1, 13]]
             }
           },
           "frames":[
              [{ # array for each frame, hash in each for each size
                 "url": "{{FRAME_BASEURL}}/{{EVENT_ID}}/{{FRAME_ID}}-{{SIZE}}.jpg",
                 "key": "frame0-original-key",
                 "size": "original",
                 "id": "DFHCjcLqMR4adGEjE7CMHh"
               }]
           ]
         }
   
   More useful when fetching large numbers of Events is the compact Event
   representation. The compact representation contains skeletal frame data
   (just enough to be able to construct the URLs yourself) and just enough
   metadata to be useful for playback.
   
   A compact Event object::

        {
           "username": "MeeSun Boice",
           "seen": 1,
           "id": "BSRKjwLq8R4adGEjE7CMHh",
           "user": 297,
           "location_name": "Half Moon Bay, CA",
           "created": 132478663322114,
           "strips": [[1, 13], [14, 18]], # in compact contains only the indexes
           "batches": [[1, 13], [14, 18]], # in compact contains only the indexes
           "frames": [ # in compact contains only the frame IDs
             "ByKKOeOYBPHImHvxW72OvX", # one for each frame
           ]
        }

   .. note::
     Any API response that includes an Event object will also have the
     ``frame_baseurl`` field in it's ``meta`` hash. This is used to
     construct the full URL for frames, batches and strips.
   
   Frame Sizes:
     All frames are stored in JPEG format and stored in a variety of sizes
     and qualities, thoughtfully optimized around various display
     requirements and bandwidth constraints:
    
     * **thumb-s** 50x50 in low quality
     * **thumb** 100x100 in medium quality
     * **small** 240x240 in low-medium quality
     * **medium** 480x480 in medium quality
     * **full** 640x640 in high quality
     * **original** original size in original quality
   
   Frames:
     Frames are strictly ordered by the order they should be displayed in
     to make sense to a viewer. Once an event has frames, no more frames
     can be added, and frames can not be removed. To construct frame URLs
     from the compact response use the following format::
     
        {meta.frame_baseurl}{items[NUM].id}/{items[NUM].frames[FRAME_NUM]}-{SIZE}.jpg
   
   Strips:
     Strips are prerendered JPEGs of frames in Events arranged
     end-to-end, with the goal of minimizing the amount of network roundtrips
     required to download an entire Event. Since there could potentially
     be many frames in an Event, strips are limited in size, therefore
     there could be multiple strips per event. 
     
     Strips are defined by their one-indexed bounds. E.g. ``small1-13.jpg`` 
     contains the small frames 1 thru 13. You can determine strip
     boundaries by the ``indexes`` field in the ``strips`` hash. Each
     two-tuple defines the boundaries of an individual strip.
     
     To construct strip URLs from the compact response use the following 
     format::

        {meta.frame_baseurl}{items[NUM].id}{SIZE}{items[NUM].strips[STRIP_NUM][0]}-{items[NUM].strips[STRIP_NUM][1]}.jpg

   Batches:
     Batches are the same idea as strips, but optimized even further for
     clients that can process binary data. The URLS are generated exactly
     the same but with the extension ``bjpeg``
   

   .. seealso::
     Streams



.. http:method:: GET events/{id}/

   :arg id: The ID of the Event to retrieve.

   Returns a single :http:response:`event-object` in the ``items`` key


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
   Streams. Additionally, only okay-ed consumers may create Events. Email
   ``support@ficture.it`` to have your consumer whitelisted.

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

        POST /api/v1/events/
        Content-Type: multipart/form-data; boundary=--asdf1
        Content-Length: 1234
        --asdf1
        Content-Disposition: form-data; name="photo-0"; filename="photo0.jpg"
        Content-Type: image/jpeg
        
        JPEG DATA....;
        --asdf1
        Content-Disposition: form-data; name="photo-0-meta"; filename="photo0.json"
        Content-Type: application/json
        
        {'some': 'metadata': ['here', 'for photo 1']}
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

