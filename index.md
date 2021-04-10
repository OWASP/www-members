---

layout: col-sidebar
title: OWASP Membership Portal (WIP)
tags: OWASP membership

---

<div id='member-qr' style='float:right;'>
</div>
<div id='member-info'>
This may take a few moments...
</div>

<script>
  $(function() {
      $.get( "https://owaspadmin.azurewebsites.net/api/get-member-info?code=mWP6TjdDSJZOQIZQNtb2fUPuzuIamwaobBZUTnN24JEdtFybiTDl7A==", { authtoken : Cookies.get('CF_Authorization') }, function( data ) {
          $('#member-info').fill_member_info(data);
          $('#member-qr').kjua({text: data['member_number']});
        }).fail(function() {
               $('#member-info').html('<strong>Failed to find member information.</strong>')
        });
  })
  
  $.fn.fill_member_info = function(data) {
        html = "Welcome, " + data['name'] + ".";
        html += "<strong>Member Number:</strong>" + data['member_number'] + "<br>";
        html += "<strong>Email:</strong>" + data['emails'][0] + "<br>";
        html += "<strong>Address:</strong>" + data['address'] + "<br>";
        html += "<strong>Phone:</strong>" + data['phone_numbers'][0] + "<br>";
        html += "<strong>Membership Type:</strong>" + data['membership_type'] + "<br>";
        html += "<strong>Membership Start:</strong>" + data['membership_start'] + "<br>";
        html += "<strong>Membership End:</strong>" + data['membership_end'] + "<br>";
        html += "<strong>Recurring:</strong>" + data['membership_recurring'] + "<br>";
        this.html(html);
    }
</script>
