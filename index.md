---

layout: col-sidebar
title: OWASP Membership Portal (WIP)
tags: OWASP membership

---

{% raw %}
<div id="membership-portal-app" style="margin: 0px;" v-cloak>
   <div id='member-info' v-if='!loading' v-model='membership_data'>
     <h3>Welcome, {{ membership_data['name'] }}</h3>
     <strong>Member Number:</strong> {{ membership_data['member_number'].substring(membership_data['member_number'].lastIndexOf('/') + 1) }}<br>
     <strong>Email:</strong>{{ membership_data['emails'][0]['email'] }}<br>
     <strong>Address:</strong>{{ membership_data['address'] }}<br>
     <strong>Phone:</strong>{{ membership_data['phone_numbers'][0]['number'] }}<br>
     <strong>Membership Type:</strong>{{ membership_data['membership_type'] }}<br>
     <strong>Membership Start:</strong>{{ membership_data['membership_start'] }}<br>
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


<!-- keep below for reference 
<div id='member-qr' style='float:right;'>
</div>
<div id='member-info'>
This may take a few moments...
<button class='cta-button' style='width:80px;height:80px;'>
  <div class='spinner'>
    <div class='inner-spinner'></div>
  </div>
</button>
</div>-->

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
    },
    created: function() {
      this.loading=true
      //might put something here eventually...
      const postData = {
        params: {
          authtoken: Cookies.get('CF_Authorization')
        }
      }
      axios.get('https://owaspadmin.azurewebsites.net/api/get-member-info?code=mWP6TjdDSJZOQIZQNtb2fUPuzuIamwaobBZUTnN24JEdtFybiTDl7A==', postData)
            .then(response => {
                this.membership_data = JSON.parse(response)
                this.loading=false
                this.$forceUpdate()
                //$('#member-info').fill_member_info(memdata);
                //$('#member-qr').kjua({text: memdata["member_number"]});
            })
            .catch(function (err) {
              this.errors = { error : 'These are not the droids you are looking for' }
              this.loading = false
            }) 
    },
  })
}, false)
</script>

<!--
<script>
  $(function() {
      $.get( "https://owaspadmin.azurewebsites.net/api/get-member-info?code=mWP6TjdDSJZOQIZQNtb2fUPuzuIamwaobBZUTnN24JEdtFybiTDl7A==", { authtoken : Cookies.get('CF_Authorization') }, function( data ) {
          memdata = JSON.parse(data)
          $('#member-info').fill_member_info(memdata);
          $('#member-qr').kjua({text: memdata["member_number"]});
        }).fail(function() {
               $('#member-info').html('<strong>You may have gotten here but currently this site only works for a limited subset of members.  Come back later.</strong>')
        });
  })
  
  $.fn.fill_member_info = function(memberdata) {
        if(memberdata) {
          html = "<h3>Welcome, " + memberdata['name'] + ".</h3><br>";
          html += "<strong>Member Number:</strong>" + memberdata['member_number'].substring(memberdata['member_number'].lastIndexOf('/') + 1) + "<br>";
          html += "<strong>Email:</strong>" + memberdata['emails'][0]['email'] + "<br>";
          html += "<strong>Address:</strong>" + memberdata['address'] + "<br>";
          html += "<strong>Phone:</strong>" + memberdata['phone_numbers'][0]['number'] + "<br>";
          html += "<strong>Membership Type:</strong>" + memberdata['membership_type'] + "<br>";
          html += "<strong>Membership Start:</strong>" + memberdata['membership_start'] + "<br>";
          html += "<strong>Membership End:</strong>" + memberdata['membership_end'] + "<br>";
          html += "<strong>Recurring:</strong>" + memberdata['membership_recurring'] + "<br>";
          this.html(html);
        } else {
          this.html('Oops.  Something wicked this way comes');
        }

    }
</script>
-->