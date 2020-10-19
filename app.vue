/*This app uses the popular VueJs library. This library makes it easier to do two-way binding and manage state*/

//start point of the app
new Vue({
  el: '#app',
  //following data elements will be bound to the html elements and displayed appropriately. They are initialized as empty if the
  //values will be determined by user action or upon application load such as areas or events.
  data() {
    return {
      //areas will be fetched from the drive bc API and rendered in select
      areas: [],
      //keeps track of areas selected by the user
      selectedAreas: [],
      //event types are pre-specified by the drive bc API.
      eventTypes: ['', 'CONSTRUCTION', 'SPECIAL_EVENT', 'INCIDENT', 'WEATHER_CONDITION'],
      //only one event type is allowed to be specified, so this variable will keep track of the selected value.
      selectedEventType: '',
      //severities are pre-specified by the API.
      severities: ['MINOR', 'MODERATE', 'MAJOR'],
      //multiple severities can be selected by the user.
      selectedSeverities: [],
      //start date variable keeps track of selected date by the user.
      startDate: '',
      //events wil be fetched from the api based on above values.
      events: [],
      //keep track of error state if there is a problem fetching events.
      errored: false,
      //this is true while fetching events and displays a "loading" notification while fetching data to the user.
      loadingEvents: false,
      //this is true while loading areas for the dropdown and displays "loading" notification to the user while fetching areas.
      loadingAreas: true
    }
  },
  mounted() {
    //as soon as the app is loaded, try to fetch areas from the drive api for the the area selection
    this.areas = [{ id: 0, name: 'Loading areas...' }];
    //use axios library to fetch data
    axios
      .get("https://api.open511.gov.bc.ca/areas?format=json")
      .then(response => (this.areas = response.data.areas))
      .catch(error => {
        this.loadingAreas = false;
        this.areas = [{ id: 0, name: 'Error. Please refresh the page.' }];
      })
      .finally(() => this.loadingAreas = false);
  },
  methods: {
    //function to extract arguments from inputs and form a URI encoded query which can be sent as part of the
    //request URI to the api.
    extractArguments() {
      var arguments = "";
      if (this.selectedSeverities.length) {
        arguments += `&severity=${encodeURI(this.selectedSeverities.join())}`;
      }
      if (this.selectedAreas.length) {
        arguments += `&area_id=${encodeURI(this.selectedAreas.join())}`;
      }
      if (this.selectedEventType) {
        arguments += `&event_type=${this.selectedEventType}`;
      }
      if (this.startDate) {
        var createdDate = encodeURI(`>=${this.startDate}T12:00:00Z`);
        arguments += `&created=${createdDate}`;
      }
      return arguments;
    },
    //triggered when the user clicks "Get Events" button in order to fetch event data from the API
    fetchEvents() {
      this.loadingEvents = true;
      var argumentStr = this.extractArguments();
      axios
        .get(`https://api.open511.gov.bc.ca/events?format=json&status=ACTIVE${argumentStr}`)
        .then((response) => {
          this.events = response.data.events;
        })
        .catch(error => {
          this.errored = true;
        })
        .finally(() => this.loadingEvents = false)
    }
  }
})