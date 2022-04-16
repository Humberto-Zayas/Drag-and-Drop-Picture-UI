<template>
  <div>
    <b-sidebar
      header-class="primary-header"
      right
      title="Tap a graphic from the library below."
      width="100%"
      id="sidebar-1"
      shadow
    >
      <div class="px-3 py-2">

       
        <transition-group
          class="list-group"
          type="transition"
          :name="'flip-list'"
          v-if="currentTopic"
        >
          <b-img-lazy
            :id="graphic.id"
            blank-src="@/assets/Iphone-spinner-1.gif"
            blank-width="32"
            blank-height="32"
            @click.native="sidebarMobileImageClicked(graphic)"
            class="list-group-item"
            v-for="graphic in currentTopic.handout_graphics"
            :key="graphic.id"
            :src="api + graphic.image_url"
          >
          </b-img-lazy>
        </transition-group>
        
        <div v-else>
          <b-skeleton-img />
        </div>
      </div>
    </b-sidebar>
    <!-- user info modal -->
    <b-modal
      title="Edit Text"
      centered
      header-bg-variant="primary"
      header-text-variant="light"
      size="lg"
      id="my-modal"
      @ok="passUserText(user_text)"
    >
      <label for="user-info-text">Text</label>
      <b-textarea
        id="user-info-text"
        placeholder="Name, Email"
        class="mt-3"
        v-model="user_text"
      />
      <template #modal-footer="{ ok, cancel }">
        <div
          style="width: 100%; padding-bottom: 1rem"
          class="d-flex justify-content-center"
        >
          <b-button size="lg" variant="primary" @click="ok()">
            Save
            <img
              class="ml-3 next"
              src="@/assets/icons/handout-builder-icons/arrow.svg"
            />
          </b-button>
          <b-button
            style="display: none"
            size="sm"
            variant="danger"
            @click="cancel()"
          >
            Cancel
          </b-button>
        </div>
        <!-- Emulate built in modal footer ok and cancel button actions -->
      </template>
    </b-modal>

    <!-- email modal -->
    <b-modal
      title="Email - Build Your Custom Handout"
      size="lg"
      header-bg-variant="primary"
      header-text-variant="light"
      id="modal-center"
      centered
    >
      <!-- Complete Email Validation -->
      <div v-if="msg">
        <b-alert
          style="text-align: center"
          class="mt-3"
          v-model="showDismissibleAlert"
          :variant="variant"
          dismissible
          >{{ msg }}
        </b-alert>
      </div>
      <b-form v-else>
        <p style="text-align: center">Confirm your email address below.</p>

        
        <label>Email *</label>

        <b-input
          style="margin: none"
          placeholder="Enter an email address"
          type="email"
         
          v-model="email"
        />
        
        <b-form-invalid-feedback id="input-live-feedback">
          Enter a valid email
        </b-form-invalid-feedback>
        <br />
        <p class="small">
          By clicking submit, I agree and acknowledge that the information I've
          provided voluntarily will be used only by Sunovion and its contracted
          third-parties to contact me regarding my request. I understand that
          Sunovion will not sell or transfer my name to any third party for
          their marketing use. Please see the most recent version of our privacy
          notice, which may change from time to time.
        </p>
        <label class="mt-3" label-size="sm">* Required</label>
      </b-form>

      <template #modal-footer="{ ok }">
        <div
          style="width: 100%; padding-bottom: 1rem"
          class="d-flex justify-content-center"
        >
          <b-button v-if="msg" size="lg" variant="primary" @click="ok()">
            Close
            <img
              class="ml-3 next"
              src="@/assets/icons/handout-builder-icons/arrow.svg"
            />
          </b-button>
          <b-button
            :disabled="!emailState"
            @click.prevent="validateEmail"
            v-else
            size="lg"
            variant="primary"
            >Submit</b-button
          >
        </div>
        <!-- Emulate built in modal footer ok and cancel button actions -->
      </template>
    </b-modal>

    <!-- Unclose-able topic selection modal -->
    <b-modal
      hide-header-close
      no-close-on-backdrop
      no-close-on-esc
      header-bg-variant="primary"
      header-text-variant="light"
      cancel-disabled
      size="lg"
      @ok="initTwo(selectedTopic)"
      id="modal-topic"
      title="Build Your Custom Handout"
      centered
    >
      <p class="text-center">
        Welcome to the Sunovion Field Medical custom handout builder. Select a
        topic area and then browse through our library of educational disease
        state graphics to construct your own personalized handout&nbsp;today!
      </p>
      <b-form-group class="mt-3">
        <label for="therapeutic-area">Therapeutic Area *</label>
        <b-form-select
          id="therapeutic-area"
          
          v-model="selected"
          :options="areaOptions"
        >
          <b-form-select-option disabled value="-1">Select a therapeutic area</b-form-select-option>
        </b-form-select>
        <div v-if="selected != null" class="mt-3">
          <label for="populated-topic">Topic *</label>
          <b-form-select
            id="populated-topic"
            v-model="selectedTopic"
            :options="mappedTopics"
          >
            <b-form-select-option disabled value="-1"
              >Select a topic</b-form-select-option
            >
          </b-form-select>
        </div>
        <div class="mt-3" v-else>
          <label for="unpopulated-topic">Topic *</label>
          <b-form-select id="unpopulated-topic" v-model="selectedTopic">
            <b-form-select-option disabled value="-1"
              >Select a topic</b-form-select-option
            >
          </b-form-select>
        </div>
        <label class="mt-3" label-size="sm">* Required</label>
      </b-form-group>
      <template #modal-footer="{ ok }">
        <div
          style="width: 100%; padding-bottom: 1rem"
          class="d-flex justify-content-center"
        >
          <b-button :disabled="selected == -1 || selectedTopic == -1" size="lg" variant="primary" @click="ok()">
            Next
            <img
              class="ml-3 next"
              src="@/assets/icons/handout-builder-icons/arrow.svg"
            />
          </b-button>
          
        </div>
        <!-- Emulate built in modal footer ok and cancel button actions -->
      </template>
    </b-modal>

    <!-- Closeable topic selection modal -->
    <b-modal
      header-bg-variant="primary"
      header-text-variant="light"
      cancel-disabled
      size="lg"
      @ok="initTwo(selectedTopic)"
      id="cancellable-modal-topic"
      title="Build Your Custom Handout"
      centered
    >
      <p class="text-center">
        Welcome to the Sunovion Field Medical custom handout builder. Select a
        topic area and then browse through our library of educational disease
        state graphics to construct your own personalized handout&nbsp;today!
      </p>
      <b-form-group class="mt-3">
        <label for="therapeutic-area">Therapeutic Area *</label>
        <b-form-select
          id="therapeutic-area"
          
          v-model="selected"
          :options="areaOptions"
        >
          <b-form-select-option disabled value="-1">Select a therapeutic area</b-form-select-option>
        </b-form-select>
        <div v-if="selected != null" class="mt-3">
          <label for="populated-topic">Topic *</label>
          <b-form-select
            id="populated-topic"
            v-model="selectedTopic"
            :options="mappedTopics"
          >
            <b-form-select-option disabled value="-1"
              >Select a topic</b-form-select-option
            >
          </b-form-select>
        </div>
        <div class="mt-3" v-else>
          <label for="unpopulated-topic">Topic *</label>
          <b-form-select id="unpopulated-topic" v-model="selectedTopic">
            <b-form-select-option disabled value="-1"
              >Select a topic</b-form-select-option
            >
          </b-form-select>
        </div>
        <label class="mt-3" label-size="sm">* Required</label>
      </b-form-group>
      <template #modal-footer="{ ok }">
        <div
          style="width: 100%; padding-bottom: 1rem"
          class="d-flex justify-content-center"
        >
          <b-button :disabled="selected == -1 || selectedTopic == -1" size="lg" variant="primary" @click="ok()">
            Next
            <img
              class="ml-3 next"
              src="@/assets/icons/handout-builder-icons/arrow.svg"
            />
          </b-button>
          
        </div>
        <!-- Emulate built in modal footer ok and cancel button actions -->
      </template>
    </b-modal>

    <!-- leftColumnImageClicked remove image confirmation -->
    <b-modal 
      ref="remove-image"
      id="remove-image-modal"
      title="Remove Image?"
      size="lg"
      header-bg-variant="primary"
      header-text-variant="light"
      centered
    >
      <p class="text-center">
        Would you like to remove this image from the poster?
      </p>

      <template #modal-footer="{ cancel }">
        <div style="width: 100%" class="d-flex justify-content-center">
          <b-button size="sm" variant="primary" @click="removeImageFromLeftColumn">
            Yes
          </b-button>
          &nbsp;&nbsp;
          <b-button size="sm" variant="primary" @click="cancel()">
            No
          </b-button>
          
          
        </div>
        <!-- Emulate built in modal footer ok and cancel button actions -->
      </template>

    </b-modal>


    <b-container fluid class="p-0">
      
      <b-overlay no-wrap :show="posterLoad" rounded="sm" />
      <template>
        <div class="app-title">
          <b-button href="/" size="sm" class="return d-none d-md-block"
            >Return To Field Medical Connect</b-button
          >
         
          <b-button href="/" size="sm" class="return d-md-none d-lg-none d-xl-none"
            >Back</b-button
          >
          <h1 style="margin: 0">Custom Handout Builder</h1>
          
          <b-button v-show="!stepThree" variant="link" class="d-md-none" v-b-toggle.sidebar-1>
            <img src="@/assets/icons/handout-builder-icons/menu.svg" />
          </b-button
          >
        </div>
      </template>

      <b-row class="flex-row" v-if="stepTwo" no-gutters>

        {{ this.$root.mobile }}
        
        <b-col md="7" class="left-col">
          <b-row no-gutters>
            <b-col>
              <div style="position: relative">
                <b-img
                  v-if="currentTopic"
                  fluid
                  :src="api + '/' + currentTopic.image_url"
                />
                <b-skeleton-img v-else no-aspect height="73px" />
                <b-button
                  class="topic-edit-button d-none d-md-none d-lg-block"
                  variant="primary"
                  size="sm"
                  v-b-modal.cancellable-modal-topic
                  >EDIT
                  <img
                    
                    src="@/assets/icons/handout-builder-icons/edit-icon.svg"
                /></b-button>
                <b-button 
                  class="topic-edit-button d-lg-none"
                  variant="primary"
                  size="sm"
                  rounded
                  v-b-modal.cancellable-modal-topic
                >
                  <img
                    
                    src="@/assets/icons/handout-builder-icons/edit-icon.svg"
                />
                </b-button>
              </div>
            </b-col>
          </b-row>
          <div class="position-relative">
            
            <draggable
              class="position-relative"
              style="z-index: 2"
              v-if="currentTopic"
              tag="div"
              v-model="list2"
              v-bind="dragOptions"
              :move="onMove"
            >
              <transition-group :class="{'pb-0': list2.length == 3}" name="no" class="list-group" tag="ul">
                <b-img-lazy
                  blank-src="@/assets/Iphone-spinner-1.gif"
                  class="list-group-item"
                  rounded="false"
                  @click.native="leftColumnImageClicked(element)"
                  v-for="element in list2"
                  :key="element.id"
                  :src="api + '/' + element.image_url"
                />
              </transition-group>
            </draggable>

            <div class="placeholder-images">
              <div
                :class="{
                  'drag-here': list2.length == 0,
                  'remove-border': true,
                }"
                class="placeholder-image-container"
              >
                <b-img
                  
                  style="width: 100%; height: 100%"
                  src="https://via.placeholder.com/3631x1091/f4f4f4/f4f4f4"
                >
                </b-img>
                <p class="large-p d-none d-md-block">Drag and drop graphic here</p>
                <div v-if="mobile" v-b-toggle.sidebar-1 class="tap-div d-md-none">
                  <div>
                    <p class="large-p">Tap here to select graphic</p>
                  </div>
                </div>
              </div>
              <div
                :class="{
                  'drag-here': list2.length == 1,
                  'remove-top-border': list2.length == 0,
                }"
                class="placeholder-image-container"
              >
                <b-img
                  
                  style="width: 100%; height: 100%"
                  src="https://via.placeholder.com/3631x1091/f4f4f4/f4f4f4"
                >
                </b-img>
                <p class="large-p d-none d-md-block">Drag and drop graphic here</p>
                <div v-if="mobile" v-b-toggle.sidebar-1 class="tap-div d-md-none">
                  <div>
                    <p class="large-p">Tap here to select graphic</p>
                  </div>
                </div>
              </div>
              <div
                :class="{
                  'drag-here': list2.length == 2,
                  'remove-top-bottom-border': list2.length == 1,
                }"
                class="placeholder-image-container"
              >
                <b-img
                  style="width: 100%; height: 100%"
                  src="https://via.placeholder.com/3631x1091/f4f4f4/f4f4f4"
                >
                </b-img>
                <p class="large-p d-none d-md-block">Drag and drop graphic here</p>
                <div v-if="mobile" v-b-toggle.sidebar-1 class="tap-div d-md-none">
                  <div>
                    <p class="large-p">Tap here to select graphic</p>
                  </div>
                </div>
              </div>
              <div class="user-info-panel">
                <b-row cols="2">
                  <b-col md="4" sm="6">
                    <div v-if="user_info">
                      <pre>
                      {{ user_info }}
                    </pre
                      >
                    </div>
                    <div v-else>
                      [Name/Practice Name]
                      <br />
                      [Email]
                    </div>
                  </b-col>
                  <b-col md="8" sm="6">
                    <p style="color: #008059; font-weight: 700">Notes:</p>
                  </b-col>
                </b-row>
                <div>
                  <b-button
                    style="position: static"
                    class="topic-edit-button mt-2"
                    variant="primary"
                    size="sm"
                    v-b-modal.my-modal
                    >Edit
                    <img
                      style="border: none"
                     
                      src="@/assets/icons/handout-builder-icons/edit-icon.svg"
                  /></b-button>
                </div>
              </div>
            </div>
          </div>
        </b-col>
        <b-col md="5" class="right-col d-none d-md-block">
          <p class="large-p">Drag a graphic from the library below.</p>

          <draggable
            v-if="currentTopic"
            tag="div"
            v-model="currentTopic.handout_graphics"
            v-bind="dragOptions"
            :move="onMove2"
            @start="isDragging = true"
            @end="isDragging = false"
          >
            <transition-group
              class="list-group"
              type="transition"
              :name="'flip-list'"
            >
              <b-img-lazy
                blank-src="@/assets/Iphone-spinner-1.gif"
                blank-width="32"
                blank-height="32"
                class="list-group-item"
                v-for="graphic in currentTopic.handout_graphics"
                :key="graphic.id"
                :src="api + graphic.image_url"
              >
              </b-img-lazy>
            </transition-group>
          </draggable>
          <div v-else>
            <b-skeleton-img />
          </div>
        </b-col>
      </b-row>
    

      <section v-if="stepThree" class="step stepThree">
        <div class="content">
          <div class="step-three-card">
            <p>
              Your custom handout is complete!<br> Email, download and print your
              handout below.
            </p>
            <b-row class="d-flex align-items-center justify-content-center">
              <iframe
                style="display: none"
                ref="myiframe"
                id="print_frames"
                src="./handout.pdf"
              />
              <br />
              <b-btn class="btn-third-step mb-2" variant="primary" v-b-modal.modal-center target="_blank">Email&nbsp;&nbsp;&nbsp;<img style="width: 14px; margin-top: -3px" src="@/assets/icons/handout-builder-icons/email-icon.svg"/></b-btn>
              &nbsp;&nbsp;&nbsp;&nbsp;

              <b-btn class="btn-third-step mb-2" variant="primary" target="_blank" :href="api + demo.pdf_url"
                >Download&nbsp;&nbsp;&nbsp;<img style="width: 14px; margin-top: -3px" src="@/assets/icons/handout-builder-icons/download-icon.svg"/></b-btn
              >
              
              &nbsp;&nbsp;&nbsp;&nbsp;
              <b-btn class="btn-third-step mb-2" variant="primary" @click="print()">Print&nbsp;&nbsp;&nbsp;<img style="width: 14px; margin-top: -3px" src="@/assets/icons/handout-builder-icons/print-icon.svg"/></b-btn>
            </b-row>
            
          </div>
        </div>
        
      </section>

     

      <div class="stepFooter">
       
        <b-button
          v-show="!stepThree"
          size="md"
          variant="primary"
          v-if="user_info && list2.length > 2"
          @click.prevent="posterConfig()"
          class="buttonStep"
          >
          Next
          <img
            class="ml-3 next"
            src="@/assets/icons/handout-builder-icons/arrow.svg"
        /></b-button>

        <b-button
          size="md"
          variant="primary"
          v-else
          disabled
          v-b-tooltip.hover
          title="Please fill out all of the required info"
          class="buttonStep"
          >Next
          <img
            class="ml-3 next"
            src="@/assets/icons/handout-builder-icons/arrow.svg"
        /></b-button>
        
        <p style="font-weight: 400; margin: 0" class="small mt-2">
          Sunovion and
          <img
            style="width: 11px; margin: -1px 2px 0"
            src="@/assets/burst@2x.png"
          />
          are registered trademarks of Sumitomo Dainippon Pharma Co., Ltd.
          <br />
          Sunovion Pharmaceuticals Inc. is a U.S. subsidiary of Sumitomo
          Dainippon Pharma Co., Ltd.
          <br />
          ©2022 Sunovion Pharmaceuticals Inc. All rights reserved.
        </p>
      </div>
    </b-container>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import config from "@/data/config"; //config.api holds endpoint for api

export default {
  name: "hello",
  components: {
    draggable,
  },
  data() {
    return {
      mobile: window.innerWidth <= 767,
      nextGraphicToBeRemoved: {},
      mainProps: { blank: true, width: 588, height: 187, class: "m1" },
      msg: null,
      variant: "primary",
      showDismissibleAlert: false,
      emailMessage: null,
      email: '',
      posterLoad: false,
      disabled: true,
      stepOne: false,
      stepTwo: true,
      stepThree: false,
      demo: null,
      api: null,
      user_info: null,
      user_text: null,
      loading: true,
      areas: null,
      areaOptions: [],
      selected: -1,
      selectedTopic: -1,
      // mappedTopics: null,
      currentTopic: null,
      list2: [],
      editable: true,
      isDragging: false,
      delayedDragging: false,
      pdfUrl: null,
      poster: {
        topic_id: null,
        top_graphic_id: null,
        middle_graphic_id: null,
        bottom_graphic_id: null,
        user_info: null,
      },
      posterCreatedID: null,
      posterCreatedResponse: null,
    };
  },
  async created() {
    // Get Areas + Topics on creation
    // GET request using fetch with async/await
    // this.$bvModal.show('remove-image-modal');
    this.posterLoad = true;
    this.api = config.api;
    const response = await fetch(config.api + "/api/therapeutic_areas");
    const data = await response.json();
    this.areas = data; // unaltered data
    const transformed = data.map(({ name, topics }, index) => ({
      value: index,
      text: name,
      topics: topics,
    }));
    this.areaOptions = transformed;
    //const defaultOption = { value: null, text: 'Select a therapeutic area' },

    //this.mapTopics(this.areaOptions[selected].topics);
    //this.initTwo(1); //load default options from api
    this.posterLoad = false;
    // this.areaOptions.unshift({
    //   value: null,
    //   text: "Select a therapeutic area",
    // });
    this.$bvModal.show("modal-topic");

    addEventListener('resize', () => {
      this.mobile = innerWidth <= 2000
    })
    

  },
  methods: {
    sidebarMobileImageClicked(graphic) { 

      const h = this.$createElement;
      const messageVNode = h('div', { class: ['foobar'] }, [
        h('p', { class: ['text-center'] }, [
          'The poster has reached the max number of images.',
          
        ]),
      ])

      console.log('Tap to Add Called: ', graphic);

      //store some data
      let list2 = this.list2; //new array for drag drop
      let origin = this.currentTopic.handout_graphics; //original array from api
      
            //manip data in array if list has room
      if (this.list2.length > 2) {
        console.log('Max reached'); 
        
        this.boxTwo = ''
        this.$bvModal.msgBoxOk([messageVNode], {
          title: 'Max Amount of Images Reached',
          size: 'lg',
          buttonSize: 'sm',
          okVariant: 'primary',
          headerBgVariant: 'primary',
          headerTextVariant: 'light',
          footerClass: 'p-2 border-top-0 justify-content-center',
          centered: true
        })
        .then(value => {
          this.boxTwo = value
        })
        .catch(err => {
          // An error occurred
        })
        
      } else {
        //filter on origin based on graphic.id
        this.currentTopic.handout_graphics = origin.filter(element => element.id !== graphic.id); //hey take everything BUT the match
        list2.push(graphic);
      }

      //origin.pop(result); // not needed
      this.$root.$emit('bv::toggle::collapse', 'sidebar-1'); //close sidebar

    },
    leftColumnImageClicked(graphic) { 

       this.$refs['remove-image'].show();
       this.nextGraphicToBeRemoved = graphic;

    },
    removeImageFromLeftColumn() {
      console.log('removeImageFromLeftColumn called');

      //store some data
      let list2 = this.list2;
      let nextGraphicToBeRemoved = this.nextGraphicToBeRemoved;

      //filter
      this.list2 = list2.filter(element => element.id !== nextGraphicToBeRemoved.id);

      // manip data
      this.currentTopic.handout_graphics.push(nextGraphicToBeRemoved);

      //close the modal
      this.$refs['remove-image'].hide();

    },
    passUserText(userText) {
      this.user_info = userText;
    },
    removeImage(relatedContext, draggedContext) {
      console.log(relatedContext);
      console.log(draggedContext);
    },
    validateEmail() {
      // validate email
      this.posterLoad = true;
      this.emailPdf();
      //setTimeout(() => this.emailPdf(), 2000);
    },
    emailPdf() {
      const email = {
        email: this.email
      }
      const id = this.posterCreatedID;

      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(email),
      };

      fetch(config.api + "/api/posters/" + id + "/mail", requestOptions)
        .then(async (response) => {
          const data = await response.json();
          this.emailMessage = data;
          // check for error response
          if (!response.ok) {
            // get error message from body or default to response status
            const error = (data && data.message) || response.status;
            return Promise.reject(error);
          }
          // Pass success message here?
          this.showDismissibleAlert = true;
          this.msg =
            "Thank you for selecting the email option. Your handout is on the\u00A0way!";
          this.variant = "success";
          this.posterLoad = false;
        })
        .catch((error) => {
          this.errorMessage = error;
          console.error("There was an error!", error);
          this.showDismissibleAlert = true;
          this.msg = "There was an error sending your email. Please try again.";
          this.variant = "danger";
        });
    },
    clearPoster() {
      //this.currentTopic.handout_graphics.concat(this.list2);
      let a = this.currentTopic.handout_graphics;
      let b = this.list2;
      b.forEach((element) => {
        a.push(element);
      });
      this.currentTopic.handout_graphics = a;
      this.list2 = [];
      this.user_info = null;
      this.editable = true;
    },
    checkEditStatus() {
      if (this.list2.length > 2) {
        //this.editable = false
        return;
      }
    },
    print() {
      //window.frames["print_frame"].print();
      const doc = this.$refs.myiframe;
      doc.contentWindow.print();
    },
    initTwo(passedSelectedTopic) {
      this.posterLoad = true;
      this.list2 = [];
      setTimeout(() => this.getTopic(passedSelectedTopic), 1000);
    },
    posterConfig() {
      this.posterLoad = true;
      // Reconfigure the data available so far to build object for POST
      // Regex user info for POST api here
      let info = this.user_info;
      let replaced = info.replace(/(\r\n|\r|\n)/g, '<br>');
      const poster = {
        topic_id: this.selectedTopic,
        top_graphic_id: this.list2[0].id, // list2[0].id
        middle_graphic_id: this.list2[1].id, // list2[1].id
        bottom_graphic_id: this.list2[2].id, //list2[2].id
        user_info: replaced
      };
      
      this.poster = poster;
      // POST req to build poster
      this.posterBuild();

      // Then GET poster with ID from response
    },
    async posterBuild() {
      const poster = this.poster;

      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(poster),
      };
      fetch(config.api + "/api/posters", requestOptions)
        .then(async (response) => {
          const data = await response.json();
          this.posterCreatedResponse = data;

          // check for error response
          if (!response.ok) {
            // get error message from body or default to response status
            const error = (data && data.message) || response.status;
            return Promise.reject(error);
          }
          this.posterCreatedID = data.id;
          //console.log("poster id:", data.id);
          setTimeout(() => this.posterGet(), 5000);
        })
        .catch((error) => {
          this.errorMessage = error;
          console.error("There was an error!", error);
        });
    },
    async posterGet() {
      const response = await fetch(
        config.api + "/api/posters/" + this.posterCreatedID
      );
      // const response = await fetch(config.api + '/api/posters/' + 6);
      const data = await response.json();
      this.demo = data;
      this.stepTwo = false;
      this.stepThree = true;
      this.posterLoad = false;

      //this.pdfUrl = data.pdf_url;
    },
    async getTopic(passedSelectedTopic) {
      // gets passed this.selectedTopic + 1 due to api strucutre the number I use for the topic is always 1 number behind

      const topic = passedSelectedTopic;
      const response = await fetch(config.api + "/api/topics/" + topic);
      const data = await response.json();
      this.currentTopic = data;
      this.disabled = true;
      this.stepTwo = true;
      this.stepOne = false;
      this.posterLoad = false;
    },
    orderList() {
      this.list = this.list.sort((one, two) => {
        return one.order - two.order;
      });
    },
    onMove({ relatedContext, draggedContext }) {
      //console.log("relatedContext:", relatedContext);
      //console.log("draggedContext", draggedContext);

      const relatedElement = relatedContext.element;

      const draggedElement = draggedContext.element;
      
      // return (
      //   (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      // );
     
    },
    onMove2({ relatedContext, draggedContext }) {
      //console.log("relatedContext:", relatedContext);
      //console.log("draggedContext", draggedContext);

      const relatedElement = relatedContext.element;

      const draggedElement = draggedContext.element;
      
      if (this.list2.length > 2) {
        return false;
      } else {
        return (
          (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
        );
      }
    },
  },
  computed: {
    mappedTopics() {
      let y = parseInt(this.selected);
      if (this.areaOptions[y]) {
        console.log('parsed int', y);
        let x = this.areaOptions[y];
        console.log('areaoptions', x);
        const transformed = x.topics.map(({ name, id }) => ({
          value: id,
          text: name,
        }));
        return transformed;
      } else {
        return []
      }
      // take selected
      // get the topics
      // return transform 
      
      //this.mappedTopics = transformed;
    },
    emailState() {
      let str = this.email
      if (str.indexOf('@') > -1)
      {
       return true 
      } else {
        return false
      }
      
     //return this.email.length > 2 ? true : false;
    },
    dragOptions() {
      return {
        animation: 250,
        group: "description",
        //disabled: this.list2.length >=3,
        disabled: !this.editable,
        ghostClass: "ghost",
      };
    },
    listString() {
      return JSON.stringify(this.currentTopic.handout_graphics, null, 2);
    },
    list2String() {
      return JSON.stringify(this.list2, null, 2);
    },
    list3String() {
      return JSON.stringify(this.poster, null, 2);
    },
  },
  watch: {
    isDragging(newValue) {
      if (newValue) {
        this.delayedDragging = true;
        return;
      }
      this.$nextTick(() => {
        this.delayedDragging = false;
      });
    },
  },
};
</script>

<style>
/* .modal-body {
  text-align: center;
} */
.tap-div {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  display: flex;
  justify-content: center;
}
.primary-header {
  background-color: #008059;
  color: white;
}
.primary-header svg {
  color: white;
}
.app-title .btn:focus {
  box-shadow: 0 0 0 0.1rem rgb(255 255 255 / 25%) !important;
}
.app-title {
  text-align: center;
  padding: 1rem;
  background: rgb(0, 128, 89);
  background: radial-gradient(
    circle,
    rgba(0, 128, 89, 1) 21%,
    rgba(0, 64, 45, 1) 100%
  );
  /* position: fixed;
    top: 0; */
  position: relative;
  width: 100%;
  /* z-index: 2; */
}
.app-title .return {
  position: absolute;
  left: 20px;
  top: 14px;
  background: rgb(255, 255, 255);
  box-shadow: inset 0px 0px 10px 5px rgb(205, 205, 205);
  /* background: radial-gradient(circle, rgba(255,255,255,1) 0%, rgba(205,205,205,1) 40%, rgba(205,205,205,1) 100%); */
  color: black;
  text-transform: uppercase;
  font-size: 14px;
  font-weight: 700;
  border-radius: 8px;
}
.app-title h1 {
  font-size: 24px;
  color: white;
  font-weight: bold;
}

.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}

.sortable-chosen {
  box-shadow: 0px 0px 0px 3px #008059 !important;
}


.list-group-item {
  cursor: move;
  padding: 0 !important;
}

.list-group-item i {
  cursor: pointer;
}

/* styles for steps */

#app {
  position: relative;
}

.flex-row {
  flex: 1;
  overflow: hidden;
}

.stepContainer {
  min-height: 92vh;
  height: auto;
  /* padding: 50px; */
  display: flex;
  flex-direction: column;
  /* justify-content: center; */
  align-items: center;
}

.step {
  /* padding: 50px;
  padding-top: 0px; */
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.content {
  margin-top: auto;
  height: 100%;
}

.buttonSteps {
  margin-top: auto;
}

.stepFooter {
  text-align: center;
  padding: 10px;
  background-color: #ffffff;
  box-shadow: 0px -5px 6px rgb(0 0 0 / 16%);
  /* position: fixed;
  bottom: 0; */
  width: 100%;
  /* z-index: 2; */
}

.stepThree {
  /* margin-top: -158px;
  margin-top: -158px; */
  /* height: 80vh; */
  justify-content: center;
  align-items: center;
  text-align: center;
}

.step-three-card {
  width: 100%;
  background-color: #eaeaea;
  padding: 3rem;
  border-radius: 7px;
  box-shadow: 0px 5px 6px rgb(0 0 0 / 16%);
}

.stepThree .content {
  max-width: 75%;
  width: 100%;
  margin: auto !important;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.left-col {
  /* max-height: 100%; */
  /* height: 140vh; */
  padding: 3rem !important;
  background-color: #f7f7f7;
  overflow: scroll;
  height: 100%;
  /* min-height: 150vh; */
}

/* .left-col span {
  min-height: 500px;
  background-color: gray;
  display: block;
} */

/* .no-gutters > [class*="col-"] {
  height: 92vh;
} */

.right-col {
  background-color: #eaeaea;
  box-shadow: inset 0px 3px 15px rgba(0, 0, 0, 0.16);
  padding: 1rem 1.5rem !important;
  overflow: scroll;
  height: 100%;
  /* height: 92vh; */
}

.placeholder-images {
  position: absolute;
  top: 0;
  width: 100%;
  z-index: 1;
}

.placeholder-image-container {
  border-left: 1px solid gray;
  border-right: 1px solid gray;
  border-top: 1px solid gray;
}

.placeholder-image-container:nth-child(3) {
  border-bottom: 1px solid gray;
}

.placeholder-image-container {
  position: relative;
}

.placeholder-image-container p {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.right-col img {
  margin-bottom: 20px;
  border: none;
  box-shadow: 0px 3px 6px rgb(0 0 0 / 16%);
}

.b-sidebar-body img {
  margin-bottom: 10px;
  border: none;
  box-shadow: 0px 3px 6px rgb(0 0 0 / 16%);
}

.user-info-panel {
  margin-bottom: 50px;
  background: rgb(221, 221, 221);
  background: linear-gradient(
    0deg,
    rgba(221, 221, 221, 1) 0%,
    rgba(255, 255, 255, 1) 30%
  );
  padding: 1.4rem;
  text-align: left;
  position: relative;
  z-index: 1;
  box-shadow: 0px 3px 6px rgb(0 0 0 / 16%);
}

.placeholder-images-container {
  box-sizing: border-box;
}

.drag-here {
  border-left: 3px dashed #008059;
  border-right: 3px dashed #008059;
  border-top: 3px dashed #008059;
  border-bottom: 3px dashed #008059 !important;
  box-sizing: border-box;
}

.remove-top-bottom-border {
  border-top: none;
}

.large-p {
  font-size: 26px;
  text-align: center;
}

pre {
  text-align: left;
  white-space: pre-line;
  font-family: "Source Sans Pro", sans-serif !important;
  font-size: 16px !important;
}

@media screen and (max-width: 1200px) {
  .large-p {
    font-size: 18px;
  }
  .topic-edit-button {
    font-size: 12px;
  }
}

@media screen and (max-width: 992px) {
  .topic-edit-button {
    bottom: 5px !important;
    right: 5px !important;
  }
  .left-col {
    padding: 2rem !important;
  }
}

@media screen and (max-width: 768px) {
  .stepThree .content {
    max-width: 90%;
  }
  .app-title h1 {
    font-size: 20px;
    text-align: center;
    width: 100%;
  }
  .large-p {
    font-size: 16px;
  }
  .app-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .app-title .return {
    position: static;
  }
  .list-group {
    min-height: 50px;
  }

}

@media screen and (max-width: 576px) {
  .left-col {
    padding: 1.25rem !important;
  }
  .topic-edit-button {
    /* bottom: 5px !important;
    right: 5px !important; */
    padding: 0px 5px !important;
  }
  pre {
    font-size: 14px;
  }

  .list-group {
    min-height: 20px;
  }
}

@media screen and (min-width: 767px) {
  .list-group {
    min-height: 200px;
    padding-bottom: 200px;
    /* height: 43%;
    background-color: rgb(181, 181, 181); */
  }
}

/* .list-group-item:last-child {
  margin-bottom: 150px;
} */

/* @media screen and (min-width: 767px) {
  .list-group {
    min-height: 120vh;
  }
} */
</style>