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
          alert( data );
          $('#member-info').fill_member_info(data['email']);
          $('#member-qr').kjua({text: data['member_number']});
        }).fail(function() {
               $('#member-info').html('<strong>Failed to find member information.</strong>')
        });
  })
  
  $.fn.fill_member_info = function(email_address) {
        html = "Welcome, " + email_address + ".  Here is your information:";
        html += "<p><strong>First Name:</strong>Test<br>";
        html += "<strong>Last Name:</strong>Leader<br>";
        html += "<strong>Member Number:</strong>8adxzfka3993dfavh<br>";
        html += "<strong>Email:</strong>test.leader@owasp.org<br>";
        html += "<strong>Email:</strong>second.email@some.place<br>";
        html += "<strong>Address:</strong>1234 Many Streets<br>";
        html += "<strong>City:</strong>Citytownville<br>";
        html += "<strong>State:</strong>Unionstate<br>";
        html += "<strong>Postal Code:</strong>534231<br";
        this.html(html);
    }
</script>
