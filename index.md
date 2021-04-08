---

layout: col-sidebar
title: OWASP Membership Portal (WIP)
tags: OWASP membership

---

<div id='member-qr' style='float:right;'></div>
<div id='member-info'>
</div>

<script>
  $(function() {
    cfauth = Cookies.get('CF_Authorization');
    if(cfauth) {
      token = getParsedJwt(cfauth);
      $('#member-info').fill_member_info(token['payload']['email']);
      $('#member-qr').kjua({text: 'https://owasp.org/manage-membership/'});
    } else {
      $('#member-info').fill_member_info('test.leader@owasp.org');
      $('#member-qr').kjua({text: 'https://owasp.org/manage-membership/'});
    }
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

  
  function getParsedJwt(strtoken) {
    token = {}
    splits = strtoken.split('.')
  try {
    token['header'] = JSON.parse(atob(splits[0]))
  } catch(e) {
  }
  try {
    token['payload'] = JSON.parse(atob(splits[1]))
  } catch (e) { 
  }
  
  try {
    token['signature'] = splits[2]
  } catch (e) { 
  }
  return token
}


</script>
