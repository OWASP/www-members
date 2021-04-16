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
  float: left;
  margin-right: 8px;
}

label {
  font-weight: bold;
  margin-right:8px;
}

button {
  padding: 16px;
}

.small {
  padding: 2px 8px;
}

.sub-item{
  margin-left: 75px;
}
</style>
{% raw %}
<div id="membership-portal-app" style="margin: 0px;" v-cloak>
   <div id='member-qr'>
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
          {{ item['number'] }}
      </template>
    </div>
    <div><button class='cta-button' v-if="mode!=1" v-on:click="switchMode">Edit Personal Information</button></div>
   </div>
   <div id='member-edit' v-if='member_ready && mode==1'>
     <label for='memname'>Name:</label><input type='text' id='memname' v-bind:value="membership_data['name']"/>
     <br>
     <label>Email:<button class='cta-button green small' v-on:click="addEmailItem()">+</button></label>
     <div class='multi-info'>
      <template v-for="item in membership_data['emails']">
          <input class='sub-item' type='text' v-bind:value="item['email']"/><button class='cta-button red small' v-on:click="removeEmailItem(item)">x</button><br>
      </template>
    </div>
    <div class='multi-info'>
      <label for="street">Street:</label><input id='street' type='text' v-bind:value="membership_data['address']['street']"/><br>
      <label for='city'>City:</label><input id='city' type='text' v-bind:value="membership_data['address']['city']"/><br>
      <label for='state'>State:</label><input id='state' type='text' v-bind:value="membership_data['address']['state']"/><br>
      <label for='postal_code'>Postal Code:</label><input id='postal_code' type='text' v-bind:value="membership_data['address']['postal_code']"/><br>
      <label for='country'>Country:</label><input id='country' type='text' v-bind:value="membership_data['address']['country']"/>
    </div>
    <label>Phone:<button class='cta-button green small' v-on:click="addPhoneItem()">+</button></label>
    <div class='multi-info'>
      <template v-for="item in membership_data['phone_numbers']">
          <input class='sub-item' type='text' v-bind:value="item['number']"/><button class='cta-button red small' v-on:click="removePhoneItem(item)">x</button><br>
      </template>
    </div>
    <div><button class='cta-button' style='padding-right:25px;' v-if="mode!=0" v-on:click="switchMode">Cancel</button><button class='cta-button green' v-if="mode!=0" v-on:click="saveInformation">Save</button></div>
   </div>
   <div id='errors' v-if="Object.keys(errors).length">
      <strong>You may have gotten here but currently this site only works for a limited subset of members.  Come back later.</strong>
   </div>
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
      errors: {},
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
                this.errors = { error : 'These are not the droids you are looking for' }
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
                */
                this.$forceUpdate()
              })
        } // end if loading
     },
     computed: {
      member_ready: function() { return (!this.loading && this.membership_data != null) }
    },
    methods:{
      switchMode: function() { 
        this.mode = !this.mode
        if(this.saved_data) {
          this.membership_data = JSON.parse(JSON.stringify(this.saved_data))
          this.saved_data = null
          //this.$forceUpdate()
        } 
      },
      removePhoneItem: function(item) {
        if(!this.saved_data) {
          this.saved_data = JSON.parse(JSON.stringify(this.membership_data))
        }
        this.membership_data['phone_numbers'].splice(this.membership_data['phone_numbers'].indexOf(item), 1)
        this.$forceUpdate()
      },
      addPhoneItem: function() {
          this.membership_data['phone_numbers'].push({'number':''})
          this.$forceUpdate()
      },
      removeEmailItem: function(item){
        if(!this.saved_data) {
            this.saved_data = JSON.parse(JSON.stringify(this.membership_data))
          }
          this.membership_data['emails'].splice(this.membership_data['emails'].indexOf(item), 1)
          this.$forceUpdate()
      },
      addEmailItem: function() {
          this.membership_data['emails'].push({'email':''})
          this.$forceUpdate()
      },
      saveInformation: function() { alert('Saved!') }
    } // end methods
  }) // end Vue
}, false) // end addEventListener
</script>
