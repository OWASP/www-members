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
</style>
{% raw %}
<div id="membership-portal-app" style="margin: 0px;" v-cloak>
   <div id='member-qr'>
   </div>
   <div id='member-info' v-if='member_ready'>
     <h3>Welcome, {{ membership_data['name'] }}</h3>
     <br>
     <strong>Member Number:</strong> {{ membership_data['member_number'].substring(membership_data['member_number'].lastIndexOf('/') + 1) }}<br>
     <template v-for="item in membership_data['emails']">
        <strong>Email:</strong>{{ item['email'] }}<br>
     </template>
     <section id='address' v-if="membership_data['address']">
      <strong>Address:</strong>{{ membership_data['address']['street'] }}<br>
      {{ membership_data['address']['city']}}, {{ membership_data['address']['state']}}&nbsp;&nbsp;{{membership_data['address']['postal code']}}
     </section>
     <template v-for="item in membership_data['phone_numbers']">
        <strong>Phone:</strong>{{ item['number'] }}<br>
     </template>
     <strong>Membership Type:</strong>{{ membership_data['membership_type'] }}<br>
     <!--<strong>Membership Start:</strong>{{ membership_data['membership_start'] }}<br>  Agree with Dawn, start has no real relevance here-->
     <strong>Membership End:</strong>{{ membership_data['membership_end'] }}<br>
     <strong>Recurring:</strong>{{ membership_data['membership_recurring'] }}<br>
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
                this.$forceUpdate()
              })
        } // end if loading
     },
     computed: {
      member_ready: function() { return (!this.loading && this.membership_data != null) }
    },
  }) // end Vue
}, false) // end addEventListener
</script>
