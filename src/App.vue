<template>
  <!-- application -->
  <div>
    <!-- ui -->
    <div class="container">
      <form @submit.prevent="requestCreate" class="was-validated row gy-2 gx-3">

        <div class="">
          <label class="visually-hidden" for="geolocation">geolocation</label>
          <input type="hidden" class="form-control" placeholder="geolocation" id="geolocation" required
            :disabled="processR" v-model="request.geolocation">
        </div>

        <div class="col-auto">
          <label class="visually-hidden" for="cityName">city</label>
          <select class="form-select" placeholder="city name" id="cityName" required :disabled="processR"
            v-model="request.cityName">
            <option v-for="city in cities" :key="city._id" v-html="city.cityName"></option>
          </select>
        </div>
        <div class="col-auto">
          <label class="visually-hidden" for="typeOfNeed">typeOfNeed</label>
          <select class="form-select" placeholder="type of need" id="typeOfNeed" required :disabled="processR"
            v-model="request.typeOfNeed">
            <option v-for="need in needs" :key="need._id" v-html="need.kind"></option>
          </select>
        </div>

        <div class="col-auto">
          <button class="btn btn-success" type="submit" :disabled="processR">Help Request</button>
        </div>

      </form>

      <main>
        <table class="table">
          <thead>
            <tr>
              <th scope="col">city</th>
              <th scope="col">volunteer</th>
              <th scope="col">need</th>
              <th scope="col">calendar</th>
              <th scope="col">geolocation</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="rvp in rvps" v-bind:key="rvp._id">
              <td v-html="rvp.cityName"></td>
              <td>
                <span class="badge bg-secondary" title="total participants" v-html="count(rvp.participants)"></span>
                <a v-on:click="selectedRVP(rvp)" href="#" data-bs-toggle="modal" data-bs-target="#modal"
                  title="be volunteer">
                  <img src="./assets/logo.png" width="32" alt="volunteer">
                </a>
              </td>
              <td v-html="rvp.typeOfNeed"></td>
              <td v-html="rvp.calendar"></td>
              <td v-html="rvp.geolocation.accuracy"></td>
            </tr>
          </tbody>
        </table>
      </main>
    </div>

    <!-- component -->

    <div class="toast-container p-3 bottom-0 end-0" id="toastPlacement" data-original-class="toast-container p-3">

      <div v-show="processR" class="toast fade show" role="alert" aria-live="assertive" aria-atomic="true">
        <div class="toast-header">
          <img src="./assets/logo.png" class="bd-placeholder-img rounded me-2" width="20" />
          <strong class="me-auto">New Request</strong>
          <small class="text-muted">5 sec ago</small>
          <button type="button" class="btn-close" data-bs-dismiss="toast" aria-label="Close"></button>
        </div>
        <div class="toast-body">
          We forward your request to our volunteers.
        </div>
      </div>

      <div v-show="processV" class="toast fade show" role="alert" aria-live="assertive" aria-atomic="true">
        <div class="toast-header">
          <img src="./assets/logo.png" class="bd-placeholder-img rounded me-2" width="20" />
          <strong class="me-auto">New volunteer</strong>
          <small class="text-muted">11 mins ago</small>
          <button type="button" class="btn-close" data-bs-dismiss="toast" aria-label="Close"></button>
        </div>
        <div class="toast-body">
          We have a new volunteer.
        </div>
      </div>

      <div v-show="processL" class="toast fade show" role="alert" aria-live="assertive" aria-atomic="true">
        <div class="toast-header">
          <img src="./assets/logo.png" class="bd-placeholder-img rounded me-2" width="20" />
          <strong class="me-auto">Location update {{ geolocation.error }}</strong>
          <small class="text-muted">{{ geolocation.speed }}</small>
          <button type="button" class="btn-close" data-bs-dismiss="toast" aria-label="Close"></button>
        </div>
        <div class="toast-body">
          {{ geolocation.latitude }} - {{ geolocation.longitude }} / {{ geolocation.accuracy }}
        </div>
      </div>
    </div>

    <div class="modal" tabindex="-1" id="modal">
      <div class="modal-dialog dialog modal-lg">
        <div class="modal-content">

          <form @submit.prevent="becomeVolunteer" class="was-validated">
            <div class="modal-header">
              <h5 class="modal-title">Become a volunteer</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close">
              </button>
            </div>
            <div class="modal-body">
              <div class="container-fluid">
                <div class="row">
                  <div class="col-8">
                    <p>Dear volunteer <b v-html="volunteer.collaborator"></b></p>
                    <p>Enter your details to fulfill the request and volunteer / <b v-html="request.typeOfNeed"></b></p>
                    <div class="form-group">
                      <label for="emailAddress" class="form-label">Email address</label>
                      <input :disabled="processV" v-model="volunteer.collaborator" type="email" class="form-control"
                        id="emailAddress" placeholder="name@example.com" required />
                      <div class="valid-feedback">Looks good!</div>
                      <div class="invalid-feedback">Please provide a valid email.</div>
                    </div>
                  </div>
                  <div class="col-4">
                    <ul class="list-group">
                      <li class="list-group-item bg-secondary text-white">Collaborators
                        <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
                          v-html="count(request.participants)">
                        </span>
                      </li>
                      <li class="list-group-item" v-for="participant in request.participants" :key="participant._id"
                        v-html="participant.volunteer_id"></li>
                    </ul>
                  </div>
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button :disabled="processV" type="submit" data-bs-dismiss="modal" class="btn btn-outline-success">
                <img src="./assets/logo.png" width="20" />
                <span>Send</span>
              </button>
            </div>
          </form>

        </div>
      </div>
    </div>

  </div>
  <!-- / application -->
</template>

<script>
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap"
import { api } from "./helpers"


export default {
  name: 'App',
  components: {

  },
  //
  data: () => ({
    //api endpoint
    a: {
      city: "/city",
      need: "/need",
      rvp: "",
      request: "/request",
      volunteer: "/volunteer",
      participant: "/participant",
    },
    // models
    // geolocation
    geolocation: {
      latitude: null,
      longitude: null,
      accuracy: null,
      altitude: null,
      altitudeAccuracy: null,
      heading: null,
      speed: null,
      timestamp: null
    },
    // request model
    request: { _id: null, cityName: null, typeOfNeed: null, geolocation: null },
    // volunter model
    volunteer: { _id: null, collaborator: null, typeOfNeed: null },
    // participant model
    participant: { _id: null, request_id: null, volunteer_id: null },
    // request process flag
    processR: false,
    // request process flag
    processV: false,
    // geopositon process flag
    processL: false,

    // default init cities
    cities: [],

    // default init needs
    needs: [],

    // default init rvps
    rvps: [],
  }),
  //
  methods: {
    async get(endpoint) {

      let response = await fetch(`${api}${endpoint}`)
      return await response.json();
    },
    async create(endpoint, data) {
      let response = await fetch(`${api}${endpoint}`, {
        method: 'POST', // *GET, POST, PUT, DELETE, etc.
        mode: 'cors', // no-cors, *cors, same-origin
        cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
        credentials: 'same-origin', // include, *same-origin, omit
        headers: {
          'Content-Type': 'application/json'
          // 'Content-Type': 'application/x-www-form-urlencoded',
        },
        redirect: 'follow', // manual, *follow, error
        referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
        body: JSON.stringify(data) // body data type must match "Content-Type" header
      });
      return await response.json();
    },

    /**
     * Count any array
     * @param {Array} data 
     */
    count(data) {
      return (data != null) ? data.length : 0
    },
    async setPostion(position) {

      this.geolocation = {
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
        accuracy: position.coords.accuracy,
        altitude: position.coords.altitude,
        altitudeAccuracy: position.coords.altitudeAccuracy,
        heading: position.coords.heading,
        speed: position.coords.speed,
        timestamp: position.timestamp
      }
      this.processL = true
      // process protection 
      setTimeout(() => {
        // done work
        this.processL = false
      }, 5000);
    },
    //
    async getGeolocation() {

      function error() {
        this.processL = true
        // process protection 
        setTimeout(() => {
          // done work
          this.processl = false
        }, 5000);
        this.geolocation.error = "Sorry, no position available"
      }

      const options = {
        enableHighAccuracy: true,
        maximumAge: 30000,
        timeout: 27000
      };

      this.watchID = navigator.geolocation.watchPosition(this.setPostion, error, options);
    },
    // 
    async init() {

      // header
      this.cities = await this.get(this.a.city);
      this.needs = await this.get(this.a.need);

      // main
      this.rvps = await this.get(this.a.rvp);

      // 
      this.getGeolocation()
    },
    // create a new request
    async requestCreate() {

      // before work
      this.processR = true
      // process protection 
      setTimeout(() => {
        // done work
        this.processR = false
      }, 5000);

      // logic
      this.request.geolocation = this.geolocation
      this.request = await this.create(`${this.a.request}/create`, this.request)
      // update main data
      this.rvps = await this.get(this.a.rvp);
    },
    async selectedRVP(rvp) {
      this.request = rvp
      this.volunteer.typeOfNeed = rvp.typeOfNeed
    },

    // create a participant
    async combineRequestAndVolunteer(participant) {

      // logic
      this.participant = await this.create(`${this.a.participant}/create`, participant)

    },

    // become a volunteer
    async becomeVolunteer() {

      // before work
      this.processV = true
      // process protection 
      setTimeout(() => {
        // done work
        this.processV = false;
      }, 5000);

      // logic
      this.volunteer.geolocation = this.geolocation
      this.volunteer = await this.create(`${this.a.volunteer}/create`, this.volunteer)
      // add logic
      // whenever a volunteer responds to the request, they join.
      this.combineRequestAndVolunteer({ request_id: this.request._id, volunteer_id: this.volunteer._id })

      // update main data
      this.rvps = await this.get(this.a.rvp);

    }
  },
  // 
  created() {
    this.init()
  }
}
</script>

<style></style>
