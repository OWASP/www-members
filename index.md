---

layout: col-sidebar
title: OWASP Membership Portal
tags: OWASP membership

---

Member Portal [TBD]


<script>
  var cfauth = Cookies.get('CF_Authorization')
  if(cfauth) {
    token = getParsedJwt(cfauth);
    alert(token);
  }
  else {
    alert(cfauth);
  }
  
  function getParsedJwt(token) {
    token = {}
    splits = token.split('.')
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
