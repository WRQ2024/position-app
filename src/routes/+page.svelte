<!-- <script> tag includes JavaScript code -->
<script>
    console.log('Script is loaded!') // testing
    import { onMount } from 'svelte'
    import Geolocation from 'svelte-geolocation' // testing
    import {
        Control,
        ControlButton,
        ControlGroup,
        DefaultMarker,
        FillLayer,
        GeoJSON,
        hoverStateFilter,
        LineLayer,
        MapEvents,
        MapLibre,
        Marker,
        Popup,
    } from 'svelte-maplibre' // DoNotChange

    /**
     * You can put functions you need for multiple components in a js file in
     * the lib folder, export them in lib/index.js and then import them like this
     */
    import { getDistance, getMapBounds } from '$lib'

    /**
     * Declare variables
     * let decalres an immutable variable
     * const declares a constant
     *
     * Note the format of markers
     */

    let markers = [
        {
            lngLat: {
                lng: 144.9638347277324,
                lat: -37.80967960080751,
            },
            label: 'Marker 1',
            name: 'This is a marker',
        },
        {
            lngLat: {
                lng: 144.96318039790924,
                lat: -37.808357984258315,
            },
            label: 'Marker 2',
            name: 'This is a marker',
        },
        {
            lngLat: {
                lng: 144.96280297287632,
                lat: -37.80668719932231,
            },
            label: 'Marker 3',
            name: 'This is a marker',
        },
    ]

    let treasures = [] // Storing Treasure Points
    let path = [] // storing the user's movement path
    // Extent of the map
    let bounds = getMapBounds(markers)

    /**
     * Declaring a function
     *
     * Functions declared in <script> can only be used in this component
     */

    function addMarker(e, label, name) {
        markers = [
            ...markers,
            {
                lngLat: e.detail.lngLat,
                label,
                name,
            },
        ]
    }

    // Geolocation API related
    const options = {
        enableHighAccuracy: true,
        timeout: Infinity, // milliseconds
        maximumAge: 0, // milliseconds, 0 disables cached positions
    }
    let getPosition = false
    let success = false
    let error = ''
    let position = {}
    let coords = []

    /**
     * $: indicates a reactive statement, meaning that this block of code is
     * executed whenever the variable used as the condition changes its value
     *
     * In this case: whenever success is set to true, a Position object
     * has been successfully obtained. Immediately update the relevant variables
     */
    $: if (success || error) {
        // reset the flag
        getPosition = false
    }

    $: if (success) {
        coords = [position.coords.longitude, position.coords.latitude]
        markers = [
            ...markers,
            {
                lngLat: { lng: coords[0], lat: coords[1] },
                label: 'Current',
                name: 'This is the current position',
            },
        ]
    }

    // Watch a position using Geolocation API if you need continuous updates
    let watchPosition = false
    let watchedPosition = {}
    let watchedMarker = {}

    /**
     * Trigger an action when getting close to a marker
     */
    let count = 0 // number of markers found
    $: if (watchedPosition.coords) { // this block is triggered when watchedPosition is updated
        // The tracked position in marker format
        watchedMarker = {
            lngLat: {
                lng: watchedPosition.coords.longitude,
                lat: watchedPosition.coords.latitude,
            },
        }

        // Whenever the watched position is updated, check if it is within 10 meters of any marker
        markers.forEach((marker) => {
            const distance = getDistance([watchedMarker, marker])

            const threshold = 10

            if (distance <= threshold) {
                count += 1
            }
        })
    }
    function generateRandomTreasures(num) {
        const newTreasures = []
        for (let i = 0; i < num; i++) {
            const lng = 144.95 + Math.random() * 0.02 // Random generation of longitude
            const lat = -37.81 + Math.random() * 0.01 // Random generation of latitude
            newTreasures.push({ lngLat: { lng, lat }, found: false, name: `Treasure ${i + 1}` })
        }
        return newTreasures
    }

    function haversine(lat1, lon1, lat2, lon2) {
        const R = 6371 // Radius of the Earth in kilometers
        const dLat = (lat2 - lat1) * Math.PI / 180
        const dLon = (lon2 - lon1) * Math.PI / 180
        const a
            = 0.5 - Math.cos(dLat) / 2 + Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * (1 - Math.cos(dLon)) / 2
        return R * 2 * Math.asin(Math.sqrt(a))
    }

    function checkForTreasure() {
        if (!position.coords || Number.isNaN(position.coords.latitude) || Number.isNaN(position.coords.longitude)) {
            console.error('Invalid user position, cannot check for treasure.')
            return
        }

        const updated = false // Checking to see if any treasure has been found

        treasures = treasures.map((treasure) => {
            const distance = haversine(
                position.coords.latitude,
                position.coords.longitude,
                treasure.lngLat.lat,
                treasure.lngLat.lng,
            )

            // If the distance is less than 50 meters, mark the treasure as found.
            if (distance < 0.05 && !treasure.found) { // 50 meters
                treasure.found = true
                console.log(`Treasure found: ${treasure.name}`)
            }
        })
    }

    /**
     * Variables can be initialised without a value and populated later
     * WARNING: this can lead to errors if the variable is used before being
     * assigned a value
     */

    let showGeoJSON = false
    let geojsonData

    /**
     * onMount is executed immediately after the component is mounted, it can be
     * used to load large datasets or to execute code required after the page
     * has been loaded
     *
     * async/await indicate an asynchronous function (i.e., program is paused
     * when a line marked with await starts and resumes when it is resolved)
     *
     * Asset files (e.g., data files, images) can be put in static folder
     *
     * Another way to load data files is to use a URL to the file hosted
     * on a remote server. Try this by replacing 'melbourne.geojson' with
     * 'https://raw.githubusercontent.com/codeforgermany/click_that_hood/main/public/data/melbourne.geojson'
     */
    onMount(async () => {
        console.log('onMount is running!') // Check if onMount is executed
        treasures = generateRandomTreasures(5) // Generate 5 random treasure spots
        console.log('Generated treasures:', treasures) // Print Generated Treasure Points
        const response = await fetch('melbourne.geojson')
        geojsonData = await response.json()
    })
</script>

<!-- Everything after <script> will be HTML for rendering -->

<!-- This section demonstrates how to get the current user location -->
<div class="flex flex-col h-[calc(100vh-80px)] w-full">
    <!-- grid, grid-cols-#, col-span-#, md:xxxx are some Tailwind utilities you can use for responsive design -->
    <div class="grid grid-cols-4">
        <div class="col-span-4 md:col-span-1 text-center">
            <h1 class="font-bold">Click button to get a one-time current position and add it to the map</h1>

            <!-- on:click declares what to do when the button is clicked -->
            <!-- In the HTML part, {} tells the framework to treat what's inside as code (variables or functions), instead of as strings -->
            <!-- () => {} is an arrow function, almost equivalent to function foo() {} -->
            <button
                class="btn btn-neutral"
                on:click={() => { getPosition = true }}
            >
                Get geolocation
            </button>

            <!-- <Geolocation> tag is used to access the Geolocation API -->
            <!-- {getPosition} is equivalent to getPosition={getPosition} -->
            <!-- bind:variable associates the parameter with the variable with the same name declared in <script> reactively -->
            <!-- let:variable creates a variable for use from the component's return -->
            <Geolocation
                {getPosition}
                options={options}
                bind:position
                let:loading
                bind:success
                bind:error
                let:notSupported
                on:position={(e) => {
                    const userPosition = e.detail // Get current user location
                    console.log('Current User Location:', userPosition.coords)
                    coords = [userPosition.coords.longitude, userPosition.coords.latitude]
                    // Updates the marker for the user's current location on the map
                    markers = [
                        ...markers,
                        { lngLat: { lng: coords[0], lat: coords[1] }, label: 'Current', name: 'Current Position' },
                    ]
                    /// Generate treasure points that do not depend on success, but are generated directly after the location is fetched
                    if (!treasures.length) { // Ensure that it is only generated once
                        treasures = generateRandomTreasures(5, coords[1], coords[0])
                        console.log('Generated Treasure:', treasures)
                    }
                }}
            >
                <!-- If-else block syntax -->
                {#if notSupported}
                    Your browser does not support the Geolocation API.
                {:else}
                    {#if loading}
                        Loading...
                    {/if}
                    {#if success}
                        Success!
                    {/if}
                    {#if error}
                        An error occurred. Error code {error.code}: {error.message}.
                    {/if}
                {/if}
            </Geolocation>

            <p class="break-words text-left">Coordinates: {coords}</p>
            <!-- Objects cannot be directly rendered, use JSON.stringify() to convert it to a string -->
            <p class="break-words text-left">Position: {JSON.stringify(position)}</p>

            <div class="text-center font-medium text-red-500">Note that in some browsers, you cannot repeatedly request the current location. If you need to continuously update the location, use the watch option below.</div>
        </div>

        <!-- This section demonstrates how to get automatically updated user location -->
        <div class="col-span-4 md:col-span-1 text-center">
            <h1 class="font-bold">Automatically updated position when moving</h1>

            <button
                class="btn btn-neutral"
                on:click={() => { watchPosition = true }}
            >
                Start watching
            </button>

            <Geolocation
                getPosition={watchPosition}
                options={options}
                watch={true}
                on:position={(e) => {
                    watchedPosition = e.detail
                    position = watchedPosition // Ensure that the location is updated
                    const newCoords = [watchedPosition.coords.longitude, watchedPosition.coords.latitude]
                    console.log('Watching position:', newCoords)
                    // Add new coordinates to the path
                    path = [...path, newCoords]
                    console.log('Updated path:', path)
                    // Check if the user is close to the treasure spot
                    checkForTreasure()
                }}
            />

            <p class="break-words text-left">watchedPosition: {JSON.stringify(watchedPosition)}</p>
        </div>

        <div class="col-span-4 md:col-span-1 text-center">
            <h1 class="font-bold">Toggle Melbourne Suburbs</h1>

            <button
                class="btn btn-neutral"
                on:click={() => { showGeoJSON = !showGeoJSON }}
            >
                Toggle
            </button>
        </div>

        <div class="col-span-4 md:col-span-1 text-center">
            <h1 class="font-bold">Found {count} markers</h1>

            The count will go up by one each time you are within 10 meters of a marker.
        </div>
    </div>

    <!-- This section demonstrates how to make a web map using MapLibre -->
    <!-- More basemap options -->
    <!-- "https://basemaps.cartocdn.com/gl/positron-gl-style/style.json" -->
    <!-- "https://tiles.basemaps.cartocdn.com/gl/dark-matter-gl-style/style.json" -->
    <!-- "https://tiles.basemaps.cartocdn.com/gl/voyager-gl-style/style.json" -->
    <MapLibre
        center={[144.97, -37.81]}
        class="map flex-grow min-h-[500px]"
        standardControls
        style="https://tiles.basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
        bind:bounds
        zoom={14}
    >
        {#if path.length > 1}
            <GeoJSON
                data={{
                    type: 'Feature',
                    geometry: {
                        type: 'LineString',
                        coordinates: path,
                    },
                }}
            >
                <LineLayer
                    layout={{ 'line-cap': 'round', 'line-join': 'round' }}
                    paint={{ 'line-color': 'blue', 'line-width': 3 }}
                    beforeLayerType="symbol"
                />
            </GeoJSON>
        {/if}
        <!-- Custom control buttons -->
        <Control class="flex flex-col gap-y-2">
            <ControlGroup>
                <ControlButton
                    on:click={() => {
                        bounds = getMapBounds(markers)
                    }}
                >
                    Fit
                </ControlButton>
            </ControlGroup>
        </Control>

        <!-- A map event to add a marker when clicked -->
        <MapEvents on:click={event => addMarker(event, 'Added', 'This is an added marker')} />

        <!-- This is how GeoJSON datasets are rendered -->
        <!-- promoteId must be a unique ID field in properties of each feature -->
        {#if showGeoJSON}
            <GeoJSON
                id="geojsonData"
                data={geojsonData}
                promoteId="name"
            >
                <FillLayer
                    paint={{
                        'fill-color': hoverStateFilter('purple', 'yellow'),
                        'fill-opacity': 0.3,
                    }}
                    beforeLayerType="symbol"
                    manageHoverState
                >
                    <Popup
                        openOn="hover"
                        let:data
                    >
                        {@const props = data?.properties}
                        {#if props}
                            <div class="flex flex-col gap-2">
                                <p>{props.name}</p>
                            </div>
                        {/if}
                    </Popup>
                </FillLayer>
                <LineLayer
                    layout={{ 'line-cap': 'round', 'line-join': 'round' }}
                    paint={{ 'line-color': 'purple', 'line-width': 3 }}
                    beforeLayerType="symbol"
                />
            </GeoJSON>
        {/if}

        <!-- Displaying markers, this is reactive -->
        <!-- For-each loop syntax -->
        <!-- markers is an object, lngLat, label, name are the fields in the object -->
        <!-- i is the index, () indicates the unique ID for each item, duplicate IDs will lead to errors -->
        {#each treasures as treasure (treasure.lngLat)}
            <Marker lngLat={treasure.lngLat}>
                <span
                    class="marker"
                    style="background-color: {treasure.found ? 'green' : 'red'};">
                    💰
                </span>
                <Popup>{treasure.name}</Popup>
            </Marker>
        {/each}
        {#each markers as { lngLat, name }, i (i)}
            <Marker {lngLat}>
                <!-- Use an image for the marker, like a treasure box icon -->
                <img
                    src="static\treasure-chest.png"
                    alt="Treasure box"
                    style="height: 40px; width: 40px;" /> <!-- Adjust size as needed -->

                <Popup
                    openOn="hover"
                    offset={[0, -10]}>
                    <div class="text-lg font-bold">{name}</div>
                </Popup>
            </Marker>
        {/each}

        <!-- Display the watched position as a marker -->
        {#if watchedMarker.lngLat}
            <DefaultMarker lngLat={watchedMarker.lngLat}>
                <Popup offset={[0, -10]}>
                    <div class="text-lg font-bold">You</div>
                </Popup>
            </DefaultMarker>
        {/if}
    </MapLibre>
</div>

<!-- Optionally, you can have a <style> tag for CSS at the end, but with TailwindCSS it is usually not necessary -->
