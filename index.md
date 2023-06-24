---

layout: col-sidebar
title: OWASP Membership Portal
tags: OWASP membership, owspmem
maintenance: true

---

<!-- rebuild 5 -->
{% if page.maintenance %}
The Membership portal is currently undergoing maintenance and will return soon. Thank you for your patience.
{% else %}
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

.info-section {
  border: 3px solid darkblue;
  border-radius: 8px;
  padding: 8px;
  margin-top: 40px;
}
.section-label {
  margin-top: -20px;
  background: white;
}

.capitalize {
    text-transform: capitalize;
}

.danger-button {
  background-color: #dc3545;
}
</style>

{% raw %}
  <div id="membership-portal-app" style="margin: 0px" v-cloak>
    <div id="member-qr"></div>
    <div id="errors" v-if="errors.length > 0">
      <label>Please correct the following:</label>
      <template v-for="(err, i) in errors">
        <template v-for="(msg, ii) in err">
          <div class="error">{{ msg }}</div>
        </template>
      </template>
    </div>
    <div
      id="member-not-found"
      v-if="!member_ready && mode == 0 && !loading && !member_logged_out"
    >
      No membership was found or your membership has expired. Please
      <a href="https://owasp.org/membership/"
        ><button class="cta-button">Join Us</button></a
      >
      <br />
      If you are a leader who has not applied for complimentary membership, you
      may do so at <a href="https://owasp.org/membership/">Become a Member</a
      ><br />
      If you feel this message is in error, submit a JIRA ticket at
      <a href="https://owasporg.atlassian.net/servicedesk/customer/portal/9/group/21/create/99">Membership Portal Support</a>
    </div>
    <div
      id="member-logged-out"
      v-if="member_logged_out && mode == 0 && !loading"
    >
      Your session has expired. Please
      <a href="https://members.owasp.org/"
        ><button class="cta-button">Log In</button></a
      >
      <br />
    </div>
    <div id="member-info" class="info-section" v-if="member_ready && mode == 0">
      <h3 class="section-label">Welcome, {{ membership_data.name }}</h3>
      <br />
      <section v-if="membership_data['member_number']">
        <div class="label">Member Number:</div>
        <div class="info">
          {{
            membership_data.member_number.substring(
              membership_data.member_number.lastIndexOf("/") + 1
            )
          }}
        </div>
      </section>
      <section v-else>
        <div class="label">Member Number:</div>
        <div class="info">
          Data not found. Contact
          <a href="mailto:membership@owasp.com">Member Services</a>
        </div>
      </section>
      <div class="label">Membership Type:</div>
      <section id="membership" v-if="membership_data.membership_type">
        <div class="info">{{ membership_data.membership_type }}</div>
        <div class="label">Membership End:</div>
        <div class="info">{{ membership_data.membership_end }}</div>
        <div v-if="renewal_near">
          <a v-bind:href="renewal_link"><button class="cta-button">Renew Now</button></a>
        </div>
        <div
          class="label"
          v-if="membership_data['membership_recurring'] == 'yes'"
        >
          Manage <a href="#">TODO: Provide link to Recurring Subscription</a>
        </div>
      </section>
      <section v-else>
        <div>No membership data found.</div>
        <a href="https://owasp.org/membership/"
          ><button class="cta-button">Renew Now</button></a
        >
      </section>
    </div>
    <!-- start email section -->
    <div id='email-info' class="info-section" v-if="member_ready && membership_data.emaillist && membership_data.emaillist.length > 1 && mode == 0">
      <h3 class="section-label">Provision an OWASP Email Address</h3>
        <div>
          Choose one from the list below:<br>
          (If you already have an OWASP email, please do not provision another)
          <hr>
        </div>
        <div v-for="error in errors">
          <label class="error-text" id="provision-error">{{error[0]}}</label>
        </div>
        <div v-for="em in membership_data.emaillist">
          <div style="display: inline-block;">
            <input type="radio" name="email_provision" v-model="chosen_email" v-bind:value="em"> &nbsp;&nbsp;{{em}}
          </div>
        </div>
        <div style="margin-top: 20px;">
          <button class="cta-button" v-on:click="redirectToAzure()" v-bind:disabled="provision_disabled">{{provision_message}}</button>
        </div>
    </div>
    <div id='email-provisioned' class="info-section" v-if="provision_email_message == true">
        <h3>Your Email was Created</h3>
        Please go to <a href="https://mail.google.com">Google Mail</a> and logout of any current account or click Add another account.  Choose 'Forgot password' and 'try another way' then 'receive a verification code'.
    </div>
    <!-- end email section -->
    <div class="info-section" v-if="member_ready && mode == 0">
      <h3 class="section-label">Personal Information</h3>
      <div class="label">Email:</div>
      <div class="multi-info">
        <template v-for="(item, i) in membership_data.emails">
          <div v-if="item.email == membership_data.membership_email" class="sub-item" >
            {{ item.email }} [Membership Email]
          </div>
          <div v-else>
            {{ item.email }} 
          </div>
        </template>
      </div>
      <div class="label">Address:</div>
      <div class="multi-info">
        <div class="sub-item">{{ membership_data.address.street }}</div>
        <div class="sub-item">{{ membership_data.address.city }}</div>
        <div class="sub-item">{{ membership_data.address.state }}</div>
        <div class="sub-item">
          {{ membership_data.address.postal_code }}
        </div>
        <div class="sub-item">{{ membership_data.address.country }}</div>
      </div>
      <div class="label">Phone:</div>
      <div class="multi-info">
        <template v-for="(item, i) in membership_data.phone_numbers">
          <div class="sub-item" >
            {{ item.number }}
          </div>
        </template>
      </div>
      <div>
        <button class="cta-button" v-if="mode != 1" v-on:click="switchMode">
          Edit Personal Information
        </button>
      </div>
    </div>
    <div id="member-edit" v-if="member_ready && mode == 1">
      <label for="memname">Name:</label
      ><input type="text" id="memname" maxlength="128" v-model="membership_data.name" />
      <br />
      <label
        >Email:<button
          class="cta-button green small"
          v-on:click="addEmailItem()"
        >
          +
        </button></label
      >
      <div class="multi-info">
        <template v-for="(item, i) in membership_data.emails">
          <input
            class="sub-item"
            type="text"
            v-model="item.email"  
            maxlength="72"          
          /><button
            class="cta-button red small"
            v-on:click="removeEmailItem(item)"
            >
            x</button
          ><br />
        </template>
      </div>
      <label for="address">Address:</label>
      <div class="multi-info" id="address">
        <label for="street">Street:</label
        ><input
          id="street"
          type="text"
          maxlength="72"
          v-model="membership_data.address.street"
        /><br />
        <label for="city">City:</label
        ><input
          id="city"
          type="text"
          maxlength="72"
          v-model="membership_data.address.city"
        /><br />
        <label for="state">State:</label
        ><input
          id="state"
          type="text"
          maxlength="72"
          v-model="membership_data.address.state"
        /><br />
        <label for="postal_code">Postal Code:</label
        ><input
          id="postal_code"
          type="text"
          maxlength="72"
          v-model="membership_data.address.postal_code"
        /><br />
        <label for="country">Country:</label>
        <select
          id="country"
          type="text"
          maxlength="72"
          v-model="membership_data.address.country"
        >
          <option value="AF">Afghanistan</option>
          <option value="AX">Åland Islands</option>
          <option value="AL">Albania</option>
          <option value="DZ">Algeria</option>
          <option value="AS">American Samoa</option>
          <option value="AD">Andorra</option>
          <option value="AO">Angola</option>
          <option value="AI">Anguilla</option>
          <option value="AQ">Antarctica</option>
          <option value="AG">Antigua and Barbuda</option>
          <option value="AR">Argentina</option>
          <option value="AM">Armenia</option>
          <option value="AW">Aruba</option>
          <option value="AU">Australia</option>
          <option value="AT">Austria</option>
          <option value="AZ">Azerbaijan</option>
          <option value="BS">Bahamas</option>
          <option value="BH">Bahrain</option>
          <option value="BD">Bangladesh</option>
          <option value="BB">Barbados</option>
          <option value="BY">Belarus</option>
          <option value="BE">Belgium</option>
          <option value="BZ">Belize</option>
          <option value="BJ">Benin</option>
          <option value="BM">Bermuda</option>
          <option value="BT">Bhutan</option>
          <option value="BO">Bolivia, Plurinational State of</option>
          <option value="BQ">Bonaire, Sint Eustatius and Saba</option>
          <option value="BA">Bosnia and Herzegovina</option>
          <option value="BW">Botswana</option>
          <option value="BV">Bouvet Island</option>
          <option value="BR">Brazil</option>
          <option value="IO">British Indian Ocean Territory</option>
          <option value="BN">Brunei Darussalam</option>
          <option value="BG">Bulgaria</option>
          <option value="BF">Burkina Faso</option>
          <option value="BI">Burundi</option>
          <option value="KH">Cambodia</option>
          <option value="CM">Cameroon</option>
          <option value="CA">Canada</option>
          <option value="CV">Cape Verde</option>
          <option value="KY">Cayman Islands</option>
          <option value="CF">Central African Republic</option>
          <option value="TD">Chad</option>
          <option value="CL">Chile</option>
          <option value="CN">China</option>
          <option value="CX">Christmas Island</option>
          <option value="CC">Cocos (Keeling) Islands</option>
          <option value="CO">Colombia</option>
          <option value="KM">Comoros</option>
          <option value="CG">Congo</option>
          <option value="CD">Congo, the Democratic Republic of the</option>
          <option value="CK">Cook Islands</option>
          <option value="CR">Costa Rica</option>
          <option value="CI">Côte d'Ivoire</option>
          <option value="HR">Croatia</option>
          <option value="CU">Cuba</option>
          <option value="CW">Curaçao</option>
          <option value="CY">Cyprus</option>
          <option value="CZ">Czech Republic</option>
          <option value="DK">Denmark</option>
          <option value="DJ">Djibouti</option>
          <option value="DM">Dominica</option>
          <option value="DO">Dominican Republic</option>
          <option value="EC">Ecuador</option>
          <option value="EG">Egypt</option>
          <option value="SV">El Salvador</option>
          <option value="GQ">Equatorial Guinea</option>
          <option value="ER">Eritrea</option>
          <option value="EE">Estonia</option>
          <option value="ET">Ethiopia</option>
          <option value="FK">Falkland Islands (Malvinas)</option>
          <option value="FO">Faroe Islands</option>
          <option value="FJ">Fiji</option>
          <option value="FI">Finland</option>
          <option value="FR">France</option>
          <option value="GF">French Guiana</option>
          <option value="PF">French Polynesia</option>
          <option value="TF">French Southern Territories</option>
          <option value="GA">Gabon</option>
          <option value="GM">Gambia</option>
          <option value="GE">Georgia</option>
          <option value="DE">Germany</option>
          <option value="GH">Ghana</option>
          <option value="GI">Gibraltar</option>
          <option value="GR">Greece</option>
          <option value="GL">Greenland</option>
          <option value="GD">Grenada</option>
          <option value="GP">Guadeloupe</option>
          <option value="GU">Guam</option>
          <option value="GT">Guatemala</option>
          <option value="GG">Guernsey</option>
          <option value="GN">Guinea</option>
          <option value="GW">Guinea-Bissau</option>
          <option value="GY">Guyana</option>
          <option value="HT">Haiti</option>
          <option value="HM">Heard Island and McDonald Islands</option>
          <option value="VA">Holy See (Vatican City State)</option>
          <option value="HN">Honduras</option>
          <option value="HK">Hong Kong</option>
          <option value="HU">Hungary</option>
          <option value="IS">Iceland</option>
          <option value="IN">India</option>
          <option value="ID">Indonesia</option>
          <option value="IR">Iran, Islamic Republic of</option>
          <option value="IQ">Iraq</option>
          <option value="IE">Ireland</option>
          <option value="IM">Isle of Man</option>
          <option value="IL">Israel</option>
          <option value="IT">Italy</option>
          <option value="JM">Jamaica</option>
          <option value="JP">Japan</option>
          <option value="JE">Jersey</option>
          <option value="JO">Jordan</option>
          <option value="KZ">Kazakhstan</option>
          <option value="KE">Kenya</option>
          <option value="KI">Kiribati</option>
          <option value="KP">Korea, Democratic People's Republic of</option>
          <option value="KR">Korea, Republic of</option>
          <option value="KW">Kuwait</option>
          <option value="KG">Kyrgyzstan</option>
          <option value="LA">Lao People's Democratic Republic</option>
          <option value="LV">Latvia</option>
          <option value="LB">Lebanon</option>
          <option value="LS">Lesotho</option>
          <option value="LR">Liberia</option>
          <option value="LY">Libya</option>
          <option value="LI">Liechtenstein</option>
          <option value="LT">Lithuania</option>
          <option value="LU">Luxembourg</option>
          <option value="MO">Macao</option>
          <option value="MK">Macedonia, the former Yugoslav Republic of</option>
          <option value="MG">Madagascar</option>
          <option value="MW">Malawi</option>
          <option value="MY">Malaysia</option>
          <option value="MV">Maldives</option>
          <option value="ML">Mali</option>
          <option value="MT">Malta</option>
          <option value="MH">Marshall Islands</option>
          <option value="MQ">Martinique</option>
          <option value="MR">Mauritania</option>
          <option value="MU">Mauritius</option>
          <option value="YT">Mayotte</option>
          <option value="MX">Mexico</option>
          <option value="FM">Micronesia, Federated States of</option>
          <option value="MD">Moldova, Republic of</option>
          <option value="MC">Monaco</option>
          <option value="MN">Mongolia</option>
          <option value="ME">Montenegro</option>
          <option value="MS">Montserrat</option>
          <option value="MA">Morocco</option>
          <option value="MZ">Mozambique</option>
          <option value="MM">Myanmar</option>
          <option value="NA">Namibia</option>
          <option value="NR">Nauru</option>
          <option value="NP">Nepal</option>
          <option value="NL">Netherlands</option>
          <option value="NC">New Caledonia</option>
          <option value="NZ">New Zealand</option>
          <option value="NI">Nicaragua</option>
          <option value="NE">Niger</option>
          <option value="NG">Nigeria</option>
          <option value="NU">Niue</option>
          <option value="NF">Norfolk Island</option>
          <option value="MP">Northern Mariana Islands</option>
          <option value="NO">Norway</option>
          <option value="OM">Oman</option>
          <option value="PK">Pakistan</option>
          <option value="PW">Palau</option>
          <option value="PS">Palestinian Territory, Occupied</option>
          <option value="PA">Panama</option>
          <option value="PG">Papua New Guinea</option>
          <option value="PY">Paraguay</option>
          <option value="PE">Peru</option>
          <option value="PH">Philippines</option>
          <option value="PN">Pitcairn</option>
          <option value="PL">Poland</option>
          <option value="PT">Portugal</option>
          <option value="PR">Puerto Rico</option>
          <option value="QA">Qatar</option>
          <option value="RE">Réunion</option>
          <option value="RO">Romania</option>
          <option value="RU">Russian Federation</option>
          <option value="RW">Rwanda</option>
          <option value="BL">Saint Barthélemy</option>
          <option value="SH">
            Saint Helena, Ascension and Tristan da Cunha
          </option>
          <option value="KN">Saint Kitts and Nevis</option>
          <option value="LC">Saint Lucia</option>
          <option value="MF">Saint Martin (French part)</option>
          <option value="PM">Saint Pierre and Miquelon</option>
          <option value="VC">Saint Vincent and the Grenadines</option>
          <option value="WS">Samoa</option>
          <option value="SM">San Marino</option>
          <option value="ST">Sao Tome and Principe</option>
          <option value="SA">Saudi Arabia</option>
          <option value="SN">Senegal</option>
          <option value="RS">Serbia</option>
          <option value="SC">Seychelles</option>
          <option value="SL">Sierra Leone</option>
          <option value="SG">Singapore</option>
          <option value="SX">Sint Maarten (Dutch part)</option>
          <option value="SK">Slovakia</option>
          <option value="SI">Slovenia</option>
          <option value="SB">Solomon Islands</option>
          <option value="SO">Somalia</option>
          <option value="ZA">South Africa</option>
          <option value="GS">
            South Georgia and the South Sandwich Islands
          </option>
          <option value="SS">South Sudan</option>
          <option value="ES">Spain</option>
          <option value="LK">Sri Lanka</option>
          <option value="SD">Sudan</option>
          <option value="SR">Suriname</option>
          <option value="SJ">Svalbard and Jan Mayen</option>
          <option value="SZ">Swaziland</option>
          <option value="SE">Sweden</option>
          <option value="CH">Switzerland</option>
          <option value="SY">Syrian Arab Republic</option>
          <option value="TW">Taiwan, Province of China</option>
          <option value="TJ">Tajikistan</option>
          <option value="TZ">Tanzania, United Republic of</option>
          <option value="TH">Thailand</option>
          <option value="TL">Timor-Leste</option>
          <option value="TG">Togo</option>
          <option value="TK">Tokelau</option>
          <option value="TO">Tonga</option>
          <option value="TT">Trinidad and Tobago</option>
          <option value="TN">Tunisia</option>
          <option value="TR">Turkey</option>
          <option value="TM">Turkmenistan</option>
          <option value="TC">Turks and Caicos Islands</option>
          <option value="TV">Tuvalu</option>
          <option value="UG">Uganda</option>
          <option value="UA">Ukraine</option>
          <option value="AE">United Arab Emirates</option>
          <option value="GB">United Kingdom</option>
          <option value="US">United States</option>
          <option value="UM">United States Minor Outlying Islands</option>
          <option value="UY">Uruguay</option>
          <option value="UZ">Uzbekistan</option>
          <option value="VU">Vanuatu</option>
          <option value="VE">Venezuela, Bolivarian Republic of</option>
          <option value="VN">Viet Nam</option>
          <option value="VG">Virgin Islands, British</option>
          <option value="VI">Virgin Islands, U.S.</option>
          <option value="WF">Wallis and Futuna</option>
          <option value="EH">Western Sahara</option>
          <option value="YE">Yemen</option>
          <option value="ZM">Zambia</option>
          <option value="ZW">Zimbabwe</option>
        </select>
      </div>
      <label
        >Phone:<button
          class="cta-button green small"
          v-on:click="addPhoneItem()"
        >
          +
        </button></label
      >
      <div class="multi-info">
        <template v-for="(item, i) in membership_data.phone_numbers">
          <!-- v-model="membership_data.phone_numbers" -->
          <input
            class="sub-item"
            type="text"
            maxlength="72"
            v-model="item.number"            
          />
          <button            
            class="cta-button red small"
            v-on:click="removePhoneItem(item)"
          >
            x</button
          ><br />
        </template>
      </div>
      <div>
        <button
          class="cta-button"
          style="padding-right: 25px"
          v-if="mode != 0"
          v-on:click="switchMode"
        >
          Cancel</button
        ><button
          class="cta-button green"
          v-if="mode != 0"
          v-on:click="saveInformation()"
        >
          Save
        </button>
      </div>
    </div>
    <!-- start leader section -->
    <div
      class="info-section"
      v-if="
        membership_data &&
        membership_data.leader_info &&
        membership_data.leader_info.length > 0 &&
        mode == 0
      "
    >
      <h3 class="section-label" >Leadership Information</h3>
      <template v-for="(item, i) in membership_data.leader_info">
        <!-- v-model="membership_data.leader_info" -->
        <div
          class="label capitalize"          
        >
          {{ item['group-type'] }} Leader
        </div>
        <div class="info" >
          <a v-bind:href="item.group_url">{{ item.group }}</a>
        </div>
      </template>
    </div>
    <!-- end leader section -->
    <!-- start billing section -->
    <div class="info-section" v-if="membership_data && membership_data.subscriptions && membership_data.subscriptions.length > 0 && mode == 0">
      <h3 class="section-label">Billing Information</h3>               
          <div v-if="membersubs.length > 0" style="margin-bottom: 40px;">
            <h3>Manage Recurring Membership</h3>
            <div v-for="membership in membersubs">
              <div><strong>{{ membership.subscription_name }}</strong></div>
              <div>{{ membership.card.brand }} ending in {{ membership.card.last_4 }}</div>
              <div>Next Billing Date: {{ membership.next_billing_date }}</div>
              <div style="margin-right: 18px; display: inline-block;">
                <button class="cta-button" v-on:click="redirectToStripe(membership.checkout_session)">Update Payment Information</button>
              </div>
              <div style="display: inline-block;">
                <button class="cta-button danger-button" v-on:click="doCancellation(membership.checkout_session)">{{ pendingCancellation === membership.checkout_session ? 'Are you sure?' : 'Cancel Recurring' }}</button>
              </div>
            </div>
          </div>
          <div v-if="donations.length > 0">
            <h3>Manage Recurring Donations</h3>
            <div v-for="donation in donations">
              <div><strong>{{ donation.subscription_name }}</strong></div>
              <div>{{ donation.card.brand }} ending in {{ donation.card.last_4 }}</div>
              <div>Next Billing Date: {{ donation.next_billing_date }}</div>
              <div style="margin-right: 18px; display: inline-block;">
                <button class="cta-button" v-on:click="redirectToStripe(donation.checkout_session)">Update Payment Information</button>
              </div>
              <div style="display: inline-block;">
                <button class="cta-button danger-button" v-on:click="doCancellation(donation.checkout_session)">{{ pendingCancellation === donation.checkout_session ? 'Are you sure?' : 'Cancel Recurring' }}</button>
              </div>
            </div>
          </div>   
    </div>
    <!-- end billing section -->


    <div id="loading" v-if="loading">
      This may take a few moments...
      <button class="cta-button" style="width: 80px; height: 80px">
        <div class="spinner">
          <div class="inner-spinner"></div>
        </div>
      </button>
    </div>
  </div>
{% endraw %}

<script src="https://js.stripe.com/v3"></script>
<script src="https://unpkg.com/vue@2"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.15/lodash.min.js"></script>
<script>
window.addEventListener('load', function() {
  new Vue({
    el: '#membership-portal-app',
    data() {
      return {
        loading: true,
        errors: [],
        membership_data: null,
        update_interval: null,
        mode: 0,
        saved_data: null,
        member_logged_out: false,
        pendingCancellation: false,
        chosen_email: '',
        provision_email_message: false,
        provision_message: 'Provision',
        provision_disabled: false
      };
    },
    created: function () {
      if (this.loading) {
        const postData = {
          params: {
            authtoken: Cookies.get("CF_Authorization"),
          },
        };
        axios
          .get(
            "https://owaspadmin.azurewebsites.net/api/get-member-info?code=mWP6TjdDSJZOQIZQNtb2fUPuzuIamwaobBZUTnN24JEdtFybiTDl7A==",
            postData
          )
          .then((response) => {
            this.membership_data = response.data;
            //Convert bad JSON to good JSON on new accounts
            if (
              typeof this.membership_data.address === "string" &&
              this.membership_data.address ===
                "{'street':'','city':'','state':'','postal_code':'','country':''}"
            ) {
              this.membership_data.address = JSON.parse(
                this.membership_data.address.replace(/'/g, '"')
              );
            } else if (typeof this.membership_data.address === "string") {
              this.membership_data.address = JSON.parse(
                this.membership_data.address
              );
            }
            this.member_logged_out = false;
            this.loading = false;

            this.$forceUpdate();
            setTimeout(
              function (membership_data) {
                if (membership_data && membership_data.name) {
                  const el = kjua({ text: membership_data.member_number });
                  const div = document.getElementById("member-qr");
                  if (div) {
                    div.appendChild(el);
                  }
                }
              },
              1000,
              this.membership_data
            );
          })
          .catch((err) => {
            this.loading = false;
            console.error(err);
            // for now assuming this is local testing
            ///*
                    this.membership_data = {}
                    this.membership_data.membership_type = 'one'
                    this.membership_data['name'] = 'Harold Test Data'
                    this.membership_data['membership_end'] = '2022-02-11'
                    this.membership_data['emails'] = [{'email':'harold.blankenship@owasp.com'},{'email':'kithwood@gmail.com'}]
                    this.membership_data.membership_email = 'harold.blankenship@owasp.com'
                    this.membership_data.phone_numbers=[{'number':'5126443053'}]
                    this.membership_data['membership_recurring']='no'
                    this.membership_data['member_number'] = 'cst_34249829348298439283749'
                    this.membership_data['address'] = {'street':'', 'city':'', 'state':'', 'postal_code':'', 'country':''}
                    this.membership_data['member-qr'] = 'https://owasp.org'
                    this.membership_data.leader_info = [{
                                                              "name": "Harold Blankenship",
                                                              "email": "harold.blankenship@owasp.com",
                                                              "group": "OWASP KLAP",
                                                              "group-type": "project",
                                                              "group_url": "https://owasp.org/www-projectchapter-example/"
                                                          },
                                                          {
                                                              "name": "Harold Blankenship",
                                                              "email": "harold.blankenship@owasp.com",
                                                              "group": "OWASP TRaP",
                                                              "group-type": "project",
                                                              "group_url": "https://owasp.org/www-projectchapter-example/"
                                                          },
                                                          {
                                                              "name": "Harold Blankenship",
                                                              "email": "harold.blankenship@owasp.com",
                                                              "group": "OWASP New Braunfels",
                                                              "group-type": "chapter",
                                                              "group_url": "https://owasp.org/www-projectchapter-example/"
                                                          }]
                    this.member_logged_out = false
                    this.loading=false
                    this.membership_data.subscriptions = [{"type":"membership", "subscription_name":"My Subscription", "card" : {"brand":"VISA", "last_4":"1234"}, "next_billing_date":"tomorrow", "checkout_session":"boo!"},{"type":"donation","subscription_name":"My Donation", "card" : {"brand":"VISA", "last_4":"1234"}, "next_billing_date":"tomorrow", "checkout_session":"Hoo!"}]
                    this.membership_data.emaillist = ["harry.b@owasp.org","b.harry@owasp.org","claptrap@owasp.org"]
                    setTimeout(function(membership_data) { 
                          if(membership_data && membership_data['name']) {
                              el = kjua({text: membership_data['member_number']});
                              div = document.getElementById('member-qr');
                              if(div) {
                                div.appendChild(el)
                              }
                          }
                      }, 1000, this.membership_data)
                      this.saved_data = JSON.parse(JSON.stringify(this.membership_data))
                  //  */

            this.$forceUpdate();
          });
      } // end if loading
    },
    computed: {
      membersubs: function () {
        return _.filter(_.get(this.membership_data, 'subscriptions', []), { type: 'membership' });
      },
      donations: function () {
        return _.filter(_.get(this.membership_data, 'subscriptions', []), { type: 'donation' });
      },
      member_ready() {
        return (
          !this.loading &&
          this.membership_data != null &&
          this.membership_data.name
        );
      },
      renewal_near() {
        if (this.membership_data.membership_end) {
          var dt = Date.parse(this.membership_data.membership_end);
          var diff = Math.abs(dt - Date.now());
          return true; // for now just show the button, regardless of if the end date is withint 30 days diff / (1000 * 60 * 60 * 24) < 30;
        } else return false;
      },
      renewal_link() {
        if(this.membership_data.membership_email)
          return "https://owasp.org/membership?email=" + this.membership_data.membership_email;
        else
          return "https://owasp.org/membership/"
      }
    },
    methods: {
      validate: function () {
        this.errors = [];

        if (this.membership_data?.name.length <= 0) {
          const nameError = { name: "Name must not be empty" };
          console.warn(nameError);
          this.errors.push(nameError);
        }

        if (this.membership_data?.name.length > 128) {
          const nameError = { name: "Name must be between 1 and 128 characters" };
          console.warn(nameError);
          this.errors.push(nameError);
        }

        if (
          !("emails" in this.membership_data) ||
          this.membership_data?.emails?.length <= 0) {
          const emailError = { email: "You must have at least one email." };
          console.warn(emailError);
          this.errors.push(emailError);
        }
        for(email in this.membership_data?.emails) {
          if(email.length <= 0 || email.length > 72){
            const emailError = { email: "Email address must be between 1 and 72 characters" };
          console.warn(emailError);
          this.errors.push(emailError);
          }
        }

        if (
          !("phone_numbers" in this.membership_data) ||
          this.membership_data?.phone_numbers?.length <= 0) {
          const phoneError = {
            phone: "You must have at least one phone number.",
          };
          console.warn(phoneError);
          this.errors.push(phoneError);
        }
        for(pnumber in this.membership_data?.phone_numbers) {
          if(pnumber.length <= 0 || pnumber.length > 72) {
            const phoneError = {
            phone: "Phone number must be between 1 and 72 characters",
          };
          console.warn(phoneError);
          this.errors.push(phoneError);
          }
        }

        if (
          !("address" in this.membership_data) ||
          this.membership_data?.address?.street?.length <= 0 ||
          this.membership_data?.address?.city?.length <= 0 ||
          this.membership_data?.address?.postal_code?.length <= 0 ||
          this.membership_data?.address?.country?.length <= 0) {
          const addressError = { address: "You must have a valid address." };
          console.warn(addressError);
          this.errors.push(addressError);
        }

        if(this.membership_data?.address?.street?.length > 72 ||
           this.membership_data?.address?.city?.length > 72 ||
           this.membership_data?.address?.postal_code?.length > 72 ||
           this.membership_data?.address?.country?.length > 72) {
             const addressError = { address: "Address fields must be between 1 and 72 characters" };
             console.warn(addressError);
             this.errors.push(addressError);
           }

        return this.errors.length === 0;
      },
      switchMode: function () {
        this.mode = !this.mode;
        if (this.saved_data) {
          this.membership_data = JSON.parse(JSON.stringify(this.saved_data));
        }
        this.errors = []; // why doesn't this set errors to empty?
        this.$forceUpdate();
        return false;
      },
      removePhoneItem: function (item) {
        this.errors = [];
        if (this.membership_data.phone_numbers?.length <= 1) {
          if (!this.errors.some((e) => e.phone)) {
            this.errors.push({
              phone: "You must have at least one phone number.",
            });
          }
          //this.errors = []
          this.$forceUpdate();
          return false;
        }

        this.membership_data.phone_numbers.splice(
          this.membership_data.phone_numbers.indexOf(item),
          1
        );
        this.$forceUpdate();
        return false;
      },
      addPhoneItem: function () {
        this.errors = [];
        if (!("phone_numbers" in this.membership_data))
          this.membership_data.phone_numbers = [];
        this.membership_data.phone_numbers.push({ number: "" });
        this.$forceUpdate();
        return false;
      },
      removeEmailItem: function (item) {
        this.errors = [];
        if (this.membership_data.emails.length <= 1) {
          if (!this.errors.some((e) => e.email)) {
            const emailError = { email: "You must have at least one email." };
            console.warn(emailError);
            this.errors.push(emailError);
          }
          //this.errors = []
          this.$forceUpdate();
          return false;
        }
        
        if (this.membership_data.membership_email == item.email) {
          if (!this.errors.some((e) => e.email)) {
            const emailError = { email: "You may not delete your membership email." };
            console.warn(emailError);
            this.errors.push(emailError);
          }
          this.$forceUpdate();
          return false;
        }

        this.membership_data.emails.splice(
          this.membership_data.emails.indexOf(item),
          1
        );
        this.$forceUpdate();
        return false;
      },
      addEmailItem: function () {
        this.errors = [];
        if (!("emails" in this.membership_data)) this.membership_data.emails = [];
        this.membership_data.emails.push({ email: "" });
        this.$forceUpdate();
        return false;
      },
      saveInformation: function () {
        this.$forceUpdate();
        if (this.validate()) {
          this.loading = true;
          const postData = {
            params: {
              authtoken: Cookies.get("CF_Authorization"),
              membership_data: this.membership_data,
            },
          };
          axios
            .post(
              "https://owaspadmin.azurewebsites.net/api/update-member-info?code=NRBl9EyVfVJYZCos5BuhquJ8KlPj/X35Isl7kNj6uk0Zr88xhPJZ5A==",
              postData
            )
            .then(() => {
              this.loading = false;
              this.mode = 0;
              this.$forceUpdate();
            })
            .catch((err) => {
              //this.errors.push({message : 'These are not the droids you are looking for' })
              this.loading = false;
              const error = { error: err };
              console.warn(error);
              this.errors.push(error);
              this.mode = 0;
              this.$forceUpdate();
            });
        }
      },
      // BELOW FUNCTIONS NEED UPDATING TO FUNCTION
      redirectToAzure: function () {
        let vm = this;
        if(!vm.chosen_email || vm.chosen_email == '')
        {
          let errors = {};
          errors.chosen_email = ['Please choose an email address.'];
          this.errors = errors;
          vm.$nextTick(function () {
                document.getElementById('provision-error').scrollIntoView();
              })
          return;
        }
        vm.provision_message = 'Please wait...(this may take some time)';
        vm.provision_disabled = true;
        const postData = {
          token: this.membership_data.customer_token,
          email: vm.chosen_email
        };
        
        axios.post('https://owaspadmin.azurewebsites.net/api/provisionemail?code=KpGlIqooyYW3GYEHuYTYzRmwSiVbeGQ4xRRarY7UWhBLwoRASFVn3g==', postData)
          .then(function (response) {
                vm.membership_data.emaillist = []
                vm.provision_email_message = true
          }).catch(function (error) {
              vm.errors = [error]
            });
  
      },
      redirectToStripe: function (sessionId) {
        stripe.redirectToCheckout({
          sessionId: sessionId
        }).then(function (result) {

        }); 
      },
      doCancellation: function (sessionId) {
        if (this.pendingCancellation && this.pendingCancellation === sessionId) {
          let vm = this;
          const postData = {
            token: sessionId
          };
          axios.post('https://owaspadmin.azurewebsites.net/api/CancelSubscription?code=Wo2wqKKpOMZP0LycmMGWLl3z8wGqK0BoIPRL/3At9W31ZnHZSRn8xw==', postData)
            .then(function (response) {
              vm.loadingUserData = true;
              vm.getMemberInfo();
            }).finally(function () {
              vm.pendingCancellation = null;
            })
        } else {
          this.pendingCancellation = sessionId;
        }
      },
    }, // end methods
  }) // end Vue
}, false) // end addEventListener
</script>
{% endif %}
