<template>
  <div>
    <!-- overlay -->
    <div class="overlay" v-if="showReglement" @click="showReglement = false"></div>

    <!-- modal -->
    <div class="modal" v-if="showReglement">
      <div class="close" @click="showReglement = false">x</div>
      <Reglement />
    </div>
    <div class="logo"><img :src="logo" width="320" /></div>
    <p class="intro">
      Je krijgt zo dadelijk 10 foto’s te zien binnen een bepaald thema. <span style="font-weight: 700; color: #f7d611">Je start met 100 seconden en verdient 10 extra
      seconden per goed antwoord</span>. Hou jij op het einde de meeste seconden over?
      <strong>Dan win je een Bol.com cadeaukaart t.w.v. 250 euro.</strong>
    </p>
    <form v-on:submit.prevent="validateLogin()">
      <div v-if="errorUserMail || errorZipCode">
        <div>
          <div class="error" v-if="errorUserMail">Gelieve een geldig e-mailadres in te vullen.</div>
          <div class="error" v-if="errorZipCode">Vul een gemeentenaam of postcode in.</div>
        </div>
      </div>

      <div>
        <div>
          <div class="error" v-if="reedsDeelgenomen === true">
            Je hebt vandaag reeds deelgenomen.<br />Speel morgen opnieuw mee.
          </div>
          <h3 class="label_title">Vul je e-mailadres in:</h3>
          <input
            class="input_form"
            type="email"
            name="MAIL"
            v-model="userMail"
            @input="(e) => (userMail = e.target.value)"
            @keyup="checkMailInCiam()"
            placeholder="jouwmail@mail.be"
            autocomplete="false"
            required
            autofocus
          />
        </div>
      </div>

      <div v-if="newUser">
        <div><p style="color: #f7d611; font-weight: 700; margin-top: 10px;" class="intro">Je bent nog niet geregistreerd. Vul onderstaande gegevens verder aan.</p></div>
      </div>

      <div v-if="newUser">
        <div>
          <h3 class="label_title">Vul je postcode of gemeente in:</h3>
          <input
            style="margin-bottom: 20px;"
            class="input_form"
            type="text"
            name="userGemeenteInput"
            v-model="userGemeenteInput"
            @keyup="checkZipCode()"
            placeholder="Bv: 2235"
            autocomplete="false"
            required
          />
          <div>
            <div
              class="autocomplete-item"
              v-for="(gemeente, index) in gemeentes"
              :key="index"
              @click="selectGemeente(gemeente.name, gemeente.zip_code)"
            >
              {{ gemeente.name }} ({{ gemeente.zip_code }})
            </div>
          </div>
        </div>
      </div>

      <div>
        <div>
          <div v-for="(optin, index) in optins" :key="index" @click="checkOptin(index)">
            <div v-show="newUser && optin.onlyForNewReg" class="optin-item">
              <div style="padding-right: 0">
                <img
                  src="https://markup.nieuwsblad.be/extra/static/2022/202210_fotoronde/img/unchecked.svg"
                  alt=""
                  class="checkbox"
                  v-if="!optin.optinChecked"
                />
                <img
                  src="https://markup.nieuwsblad.be/extra/static/2022/202210_fotoronde/img/checked.svg"
                  alt=""
                  class="checkbox"
                  v-if="optin.optinChecked"
                />
              </div>
              <div style="color: white" :ref="optin.optinName">
                {{ optin.optinTekst }}
              </div>
            </div>

            <div v-show="!optin.onlyForNewReg" class="optin-item">
              <div style="padding-right: 0">
                <img
                  src="https://markup.nieuwsblad.be/extra/static/2022/202210_fotoronde/img/unchecked.svg"
                  alt=""
                  class="checkbox"
                  v-if="!optin.optinChecked"
                />
                <img
                  src="https://markup.nieuwsblad.be/extra/static/2022/202210_fotoronde/img/checked.svg"
                  alt=""
                  class="checkbox"
                  v-if="optin.optinChecked"
                />
              </div>
              <div>
                <span v-html="optin.optinTekst"></span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div>
        <div>
          <button @click="validateFields()">START DE QUIZ</button>
        </div>
      </div>
    </form>
    <div class="footer">
      <div @click="showReglement = true">Reglement</div>
    </div>
  </div>
</template>

<script>
import Reglement from './Reglement.vue';
const url_ciam_check =
  'https://interactief2.nieuwsblad.be/WedstrijdCR/WedstrijdCR.aspx?ID=w_1wXmbiOeZPMl89jlDTxbU126qvQGXjwhQXVBSG4sBve8jMM2acagHcJFMPn5Yj0n1rzBV3eeVDlw4AQB&notags=1&mail=';

export default {
  name: 'RegistratieLight',
  props: {
    optins: Array,
    inputGemeente: String,
  },
  components: {
    Reglement,
  },
  data() {
    return {
      logo: 'https://static.nieuwsblad.be/Assets/Images_Upload/2022/10/17/fotoronde_footer.png',
      /****CIAM*****/
      userMail: '',
      newUser: false,
      userGemeenteInput: '',
      zip_code: '',
      gemeentes: [],
      city: '',
      showCity: false,
      errorUserMail: false,
      errorZipCode: false,
      reedsDeelgenomen: false,
      /////OPTINS
      /// PM EN REGIO HEBBEN ZIPCODE NODIG!!!!
      //Snel denken: ofwel iets met die methode die Jimmy gebruikt in optinbox flow, ofwel hard-coded optins toevoegen aan de Ajax-post.
      optinsChecked: [],
      showReglement: false,
    };
  },
  mounted() {
    if (localStorage.hasOwnProperty('EMAIL_202210_FOTORONDE')) {
      var maillocalstorage = localStorage.getItem('EMAIL_202210_FOTORONDE');
      this.userMail = maillocalstorage;
    }
  },

  methods: {
    checkMailInCiam: function () {
      function simIsValidEmailAddress(emailAddress) {
        //eslint-disable-next-line
        var pattern = new RegExp(
          /^((([a-z]|\d|[!#\$%&'\*\+\-\/=\?\^_`{\|}~]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])+(\.([a-z]|\d|[!#\$%&'\*\+\-\/=\?\^_`{\|}~]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])+)*)|((\x22)((((\x20|\x09)*(\x0d\x0a))?(\x20|\x09)+)?(([\x01-\x08\x0b\x0c\x0e-\x1f\x7f]|\x21|[\x23-\x5b]|[\x5d-\x7e]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])|(\\([\x01-\x09\x0b\x0c\x0d-\x7f]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF]))))*(((\x20|\x09)*(\x0d\x0a))?(\x20|\x09)+)?(\x22)))@((([a-z]|\d|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])|(([a-z]|\d|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])([a-z]|\d|-|\.|_|~|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])*([a-z]|\d|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])))\.)+(([a-z]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])|(([a-z]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])([a-z]|\d|-|\.|_|~|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])*([a-z]|[\u00A0-\uD7FF\uF900-\uFDCF\uFDF0-\uFFEF])))\.?$/i
        );

        return pattern.test(emailAddress);
      }

      if (simIsValidEmailAddress(this.userMail)) {
        var self = this;
        setTimeout(function () {
          var url = url_ciam_check + self.userMail;
          var xhr = new XMLHttpRequest();
          xhr.open('GET', url);
          xhr.onload = function () {
            if (xhr.status === 200) {
              //console.log('User\'s name is ' + xhr.responseText);
              if (xhr.responseText.indexOf('NOT_FOUND') >= 0) {
                console.log('Email not found');
                self.newUser = true;
                //self.checkBox()
              } else {
                console.log('Email found');
                self.newUser = false;
                //self.checkBox()
              }
            } else {
              console.log('Request failed.  Returned status of ' + xhr.status);
            }
          };
          xhr.send();
        }, 500);
      }
    },

    checkZipCode: function () {
      const url_zipcode_check =
        'https://interactief2.nieuwsblad.be/WedstrijdCR/WedstrijdCR.aspx?ID=ImrsUUT17SyQumFQAEtlotww8kMSfobYJzUj3f7zkC%2Bk%2BF0SH_3jpKGWcZ0PW7gPmEzyd_e4xEVKd4&notags=1&NAME=' +
        this.userGemeenteInput;

      if (this.userGemeenteInput.length > 1) {
        fetch(url_zipcode_check)
          .then((response) => response.json())
          .then((data) => {
            this.gemeentes = data.cities.city;
            this.city = data.cities.city[0].name;
            this.zip_code = data.cities.city[0].zip_code;
          });
      }
      if (this.userGemeenteInput.length <= 1) {
        this.gemeentes = [];
      }
    },

    selectGemeente: function (gemeentenaam, hoofdpostcode) {
      this.city = gemeentenaam;
      this.zip_code = hoofdpostcode;

      const userGemeenteInput = this.userGemeenteInput;
      function hasNumber(userGemeenteInput) {
        return /\d/.test(userGemeenteInput);
      }
      if (hasNumber(userGemeenteInput) == true) {
        this.userGemeenteInput = this.zip_code;
      } else {
        this.userGemeenteInput = this.city;
      }

      this.gemeentes = [];
    },

    checkOptin: function (index) {
      this.optins[index].optinChecked = !this.optins[index].optinChecked;
    },

    validateFields: function () {
      if (this.userMail == '') {
        this.errorUserMail = true;
      }
      if (this.newUser && this.zip_code == '') {
        this.errorZipCode = true;
      }
    },

    validateLogin: function () {
      console.log(this.userMail);
      console.log(this.zip_code);
      console.log(this.city);

      if (this.userMail == '') {
        this.errorUserMail = true;
      }
      if (this.newUser && this.zip_code == '') {
        this.errorZipCode = true;
      }

      this.userDataToParent();
    },

    userDataToParent: function () {
      var self = this;
      var optins = this.optins.filter(function (optin) {
        return optin.optinChecked == true;
      });

      const userData = { mail: this.userMail, zip_code: this.zip_code, optins: optins };

      var postToMessagentUrl =
        'https://interactief2.nieuwsblad.be/WedstrijdCR/WedstrijdCR.aspx?ID=fM1JgpaZGgAVWJuY9yuQwT3EibFwDNhhaXDjcFsyOkglnFGj6v8zCkPb5XCplJhYltP111lnhRqrsf&notags=1&MAIL=';

      var postToMessagentUrlWithParams =
        postToMessagentUrl + this.userMail + '&zip_code=' + this.zip_code + '&city=' + this.city;

      for (var i = 0; i < optins.length; i++) {
        console.log(optins[i].optinName);
        postToMessagentUrlWithParams = postToMessagentUrlWithParams + '&' + optins[i].optinName + '=1';
      }

      localStorage.setItem("EMAIL_202210_FOTORONDE", self.userMail);

      console.log(postToMessagentUrlWithParams);

      fetch(postToMessagentUrlWithParams)
        .then((response) => response.text())
        .then((response) => {
          response = response.replace(/(\r\n|\n|\r)/gm, '');

          if (response === 'reedsdeelgenomen') {
            this.reedsDeelgenomen = true;
          } else {
            this.reedsDeelgenomen = false;

            this.$emit('userDataFromParent', userData);
            this.$emit('registratieIsOK', true);
          }
        });
        
    },
  },
};
</script>

<style lang="css" scoped>
.autocomplete-item {
  background: white;
  padding: 10px;
  border-bottom: 1px solid #efefef;
  width: 80%;
  margin: 0 auto;
}
img.checkbox {
  width: 22px;
  margin-top: 5px;
}
.optin-item {
  font-size: 1rem;
  line-height: 1.4rem;
  display: flex;
  width: 80%;
  margin: 0 auto;
}
</style>
