---

layout: col-sidebar
title: OWASP Membership Portal (WIP)
tags: OWASP membership

---

<!-- rebuild 3 -->

<style>
[v-cloak] {display: none}

#member-qr {
  float:right;
  padding: 16px;
}

.label {
  font-weight: bold;
  margin-right: 8px;
}

.info, .multi-info {
  margin-bottom:16px;
  margin-left: 75px;
}

label {
  font-weight: bold;
  margin-right:8px;
}

button {
  margin-right: 16px;
}

.small {
  padding: 2px 8px;
}

.errors {
  padding-bottom: 24px;
  padding-top: 12px;
  border-top: 3px dotted red;
}
.error {
  font-weight:bold;
  color: darkred;
  border-left: 5px solid red;
  padding-left: 8px;
}
</style>
{% raw %}
<div id="membership-portal-app" style="margin: 0px;" v-cloak>
   <div id='member-qr'>
   </div>
   <div id='errors' v-if='errors.length > 0'>
        <label>Please correct the following:</label>
        <template v-for='err in errors'>
              <template v-for='(value, name) in err'>
                <div class='error'>{{value}}</div>
              </template>
        </template>
     </div>
   <div id='member-info' v-if='member_ready && mode==0'>
     <h3>Welcome, {{ membership_data['name'] }}</h3>
     <br>
     <div class='label'>Member Number:</div><div class='info'>{{ membership_data['member_number'].substring(membership_data['member_number'].lastIndexOf('/') + 1) }}</div>
     <div class='label'>Membership Type:</div>
     <div class='info'>{{ membership_data['membership_type'] }}</div>
     <!--<strong>Membership Start:</strong>{{ membership_data['membership_start'] }}<br>  Agree with Dawn, start has no real relevance here-->
     <div class='label'>Membership End:</div><div class='info'>{{ membership_data['membership_end'] }}</div>
     <div class='label'>Recurring:</div><div class='info'>{{ membership_data['membership_recurring'] }}</div>
     <div class='label'>Email:</div>
     <div class='multi-info'>
      <template v-for="item in membership_data['emails']">
          <div class='sub-item'>{{ item['email'] }}</div>
      </template>
    </div>
    <div class='label'>Address:</div>
    <div class='multi-info'>
      <div class='sub-item'>{{ membership_data['address']['street'] }}</div>
      <div class='sub-item'>{{ membership_data['address']['city']}}</div>
      <div class='sub-item'>{{ membership_data['address']['state']}}</div>
      <div class='sub-item'>{{membership_data['address']['postal_code']}}</div>
      <div class='sub-item'>{{membership_data['address']['country']}}</div>
    </div>
    <div class='label'>Phone:</div>
    <div class='multi-info'>
      <template v-for="item in membership_data['phone_numbers']">
          <div class='sub-item'>{{ item['number'] }}</div>
      </template>
    </div>
    <div><button class='cta-button' v-if="mode!=1" v-on:click="switchMode">Edit Personal Information</button></div>
   </div>
   <form class="form-container" v-on:submit.prevent="saveInformation">
   <div id='member-edit' v-if='member_ready && mode==1'>
     <label for='memname'>Name:</label><input type='text' id='memname' v-model="membership_data['name']"/>
     <br>
     <label>Email:<button class='cta-button green small' v-on:click="addEmailItem()">+</button></label>
     <div class='multi-info'>
      <template v-for="item in membership_data['emails']" v-model="membership_data['emails']">
          <input class='sub-item' type='text' v-model="item['email']"/><button class='cta-button red small' v-on:click="removeEmailItem(item)">x</button><br>
      </template>
    </div>
    <label for='address'>Address:</label>
    <div class='multi-info' id='address'>
      <label for="street">Street:</label><input id='street' type='text' v-model="membership_data['address']['street']"/><br>
      <label for='city'>City:</label><input id='city' type='text' v-model="membership_data['address']['city']"/><br>
      <label for='state'>State:</label><input id='state' type='text' v-model="membership_data['address']['state']"/><br>
      <label for='postal_code'>Postal Code:</label><input id='postal_code' type='text' v-model="membership_data['address']['postal_code']"/><br>
      <label for='country'>Country:</label><input id='country' type='text' v-model="membership_data['address']['country']"/>
    </div>
    <label>Phone:<button class='cta-button green small' v-on:click="addPhoneItem()">+</button></label>
    <div class='multi-info'>
      <template v-for="item in membership_data['phone_numbers']" v-model="membership_data['phone_numbers']">
          <input class='sub-item' type='text' v-model="item['number']"/><button class='cta-button red small' v-on:click="removePhoneItem(item)">x</button><br>
      </template>
    </div>
    <div><button class='cta-button' style='padding-right:25px;' v-if="mode!=0" v-on:click="switchMode">Cancel</button><button type='submit' class='cta-button green' v-if="mode!=0">Save</button></div>
   </div>
   </form>
   <div id='loading' v-if='loading'>
      This may take a few moments...
      <button class='cta-button' style='width:80px;height:80px;'>
        <div class='spinner'>
          <div class='inner-spinner'></div>
        </div>
      </button>
   </div>
</div>
{% endraw %}

<script src="https://js.stripe.com/v3"></script>
<script src="https://unpkg.com/vue"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

<script>
window.addEventListener('load', function() {
  new Vue({
    el: '#membership-portal-app',
    data: {
      loading: true,
      errors: [],
      membership_data: null,
      update_interval : null,
      mode: 0,
      saved_data: null,
    },
    created: function() {
        if(this.loading){
            const postData = {
            params: {
                authtoken: Cookies.get('CF_Authorization')
              }
            }
            axios.get('https://owaspadmin.azurewebsites.net/api/get-member-info?code=mWP6TjdDSJZOQIZQNtb2fUPuzuIamwaobBZUTnN24JEdtFybiTDl7A==', postData)
              .then(response => {
                  this.membership_data = response.data
                  this.loading=false
                 
                  this.$forceUpdate()
                  setTimeout(function(membership_data) { 
                      if(membership_data) {
                          el = kjua({text: membership_data['member_number']});
                          div = document.getElementById('member-qr');
                          if(div) {
                            div.appendChild(el)
                          }
                      }
                  }, 1000, this.membership_data)
                  //$('#member-qr').kjua({text: memdata["member_number"]});
              })
              .catch(err => {
                this.errors.push({message : err })
                this.loading = false
                // for now assuming this is local testing
                /*
                this.membership_data = {}
                this.membership_data['membership_type'] = 'one'
                this.membership_data['name'] = 'Harold Test Data'
                this.membership_data['membership_end'] = '2022-03-22'
                this.membership_data['emails'] = [{'email':'harold.blankenship@owasp.com'},{'email':'kithwood@gmail.com'}]
                this.membership_data['phone_numbers']=[{'number':'5126443053'}]
                this.membership_data['membership_recurring']='no'
                this.membership_data['member_number'] = 'owasp.org'
                this.membership_data['address'] = {'street':'123 street', 'city':'My City', 'state':'My State', 'postal_code':'12345', 'country':'My Country'}
                this.membership_data['member-qr'] = 'https://owasp.org'
                
                setTimeout(function(membership_data) { 
                      if(membership_data) {
                          el = kjua({text: membership_data['member_number']});
                          div = document.getElementById('member-qr');
                          if(div) {
                            div.appendChild(el)
                          }
                      }
                  }, 1000, this.membership_data)
                  this.saved_data = JSON.parse(JSON.stringify(this.membership_data))
                */
                
                this.$forceUpdate()
              })
        } // end if loading
     },
     computed: {
      member_ready: function() { return (!this.loading && this.membership_data != null) }
    },
    methods:{
      validate: function () {
        if(this.membership_data['name'].length <= 0) {
          error = { 'name':'Name must not be empty'}
          this.errors.push(error)
        }
        if(this.membership_data['emails'].length <= 0) {
          error = { 'email':'You must have at least one email.'}
          this.errors.push(error)
        }
        if(this.membership_data['phone_numbers'].length <= 0) {
          error = { 'email':'You must have at least one phone number.'}
          this.errors.push(error)
        }
        if(this.membership_data['address']['street'].length <= 0 ||
          this.membership_data['address']['city'].length <= 0 ||
          this.membership_data['address']['state'].length <= 0 ||
          this.membership_data['address']['postal_code'].length <= 0 ||
          this.membership_data['address']['country'].length <= 0) {

          error = { 'address':'Address must be complete.'}
          this.errors.push(error) 
        }

        return this.errors.length == 0
      },
      switchMode: function() { 
        this.mode = !this.mode
        if(this.saved_data) {
          this.membership_data = JSON.parse(JSON.stringify(this.saved_data))
        }
        this.errors = [] // why doesn't this set errors to empty?
        this.$forceUpdate()
        return false;
      },
      removePhoneItem: function(item) {
        if(this.membership_data['phone_numbers'].length <= 1) {
          error = { phone :'You must have at least one phone number.' }
          if(!this.errors.some(e => e.phone)) {
            this.errors.push(error)
          }
          this.$forceUpdate()
          return false;
        }
        
        this.membership_data['phone_numbers'].splice(this.membership_data['phone_numbers'].indexOf(item), 1)
        this.$forceUpdate()
        return false;
      },
      addPhoneItem: function() {
          this.errors = []
          this.membership_data['phone_numbers'].push({'number':''})
          this.$forceUpdate()
          return false;
      },
      removeEmailItem: function(item){
        if(this.membership_data['emails'].length <= 1) {
          error = { email :'You must have at least one email.' }
           if(!this.errors.some(e => e.email)) {
            this.errors.push(error)
          }
          this.$forceUpdate()
          return false;
        }

          this.membership_data['emails'].splice(this.membership_data['emails'].indexOf(item), 1)
          this.$forceUpdate()
          return false;
      },
      addEmailItem: function() {
          this.errors = []
          this.membership_data['emails'].push({'email':''})
          this.$forceUpdate()
          return false;
      },
      saveInformation: function() {
        this.$forceUpdate() 
        if(this.validate()){
          this.loading=true
          const postData = {
            params: {
                authtoken: Cookies.get('CF_Authorization'),
                membership_data: this.membership_data
              }
            }
          axios.get('https://owaspadmin.azurewebsites.net/api/update-member-info?code=NRBl9EyVfVJYZCos5BuhquJ8KlPj/X35Isl7kNj6uk0Zr88xhPJZ5A==', postData)
              .then(response => {
                  this.loading=false
                  this.mode = 0
                  this.$forceUpdate()
              })
              .catch(err => {
                //this.errors.push({message : 'These are not the droids you are looking for' })
                this.loading = false
                error = { 'error':err }
                this.errors.push(error)
                this.mode = 0
                this.$forceUpdate()
              })
        } 
      }
    } // end methods
  }) // end Vue
}, false) // end addEventListener
</script>
