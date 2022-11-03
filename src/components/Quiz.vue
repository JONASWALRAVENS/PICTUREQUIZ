<script>
import Share from './Share.vue';
import RegistratieLight from './RegistratieLight.vue';
import Reglement from './Reglement.vue';

var date = new Date();
var day = date.getDate();

export default {
  data() {
    return {
      currentScreen: 0,
      optins: [
        // {
        //     optinShortname: "Namiddag Update", // voor deze optin moet postcode gevraagd worden
        //     optinName: "REG_OPTIN_PM",
        //     optinServCode: "SERV_AC_PM_NEWS",
        //     optinTekst: "Ja, ik wil graag de dagelijkse namiddagupdate van het Nieuwsblad ontvangen. Je ontdekt er als eerste of jouw favoriete frituur <strong>De Beste Frituur van gemeente</strong> wordt.",
        //     optinActive: true,
        //     onlyForNewReg: true,
        //     optinChecked: false
        // },

        // {
        //     optinShortname: "Regio",
        //     optinName: "REG_OPTIN_REGIO",
        //     optinServCode: "SERV_AC_NB_REGIO",
        //     optinTekst: "Ja, ik wil op de hoogte blijven van het nieuws uit mijn regio.",
        //     optinActive: true,
        //     onlyForNewReg: false,
        //     optinChecked: false
        // },
        {
          optinShortname: 'Commercial',
          optinName: 'REG_OPTIN_COMMERCIAL',
          optinServCode: 'SERV_AC_NB_COMMERCIAL',
          optinTekst: 'Ja, ik wil graag via mail acties en aanbiedingen van het Nieuwsblad ontvangen.',
          optinActive: true,
          onlyForNewReg: true,
          optinChecked: false,
        },
        {
          optinShortname: 'Shop',
          optinName: 'REG_OPTIN_WEBSHOP',
          optinServCode: 'SERV_AC_NB_WEBSHOP',
          optinTekst:
            'Ik ontvang de nieuwsbrief van Nieuwsbladshop.be: elke week acties  op tickets, elektro, boeken,...',
          optinActive: true,
          onlyForNewReg: true,
          optinChecked: false,
        },
        // {
        //     optinShortname: "NB Partner",
        //     optinName: "REG_OPTIN_PARTNER",
        //     optinServCode: "SERV_AC_NB_PARTNER",
        //     optinTekst: "Ik ontvang graag interessante promoties en acties van partners van het Nieuwsblad.",
        //     optinActive: true,
        //     onlyForNewReg: true,
        //     optinChecked: false
        // },
        // {
        //     optinShortname: "Actie Nostalgie",
        //     optinName: "REG_OPTIN_ACTIE_NOSTALGIE",
        //     optinServCode: "",
        //     optinTekst: "Ja, hou me via mail op de hoogte van acties en aanbiedingen van Nostalgie.",
        //     optinActive: true,
        //     onlyForNewReg: false,
        //     optinChecked: false
        // },

        // {
        //     optinShortname: "Actie Mora",
        //     optinName: "REG_OPTIN_ACTIE_MORA",
        //     optinServCode: "",
        //     optinTekst: "Ja, hou me via mail op de hoogte van acties en aanbiedingen van Mora.",
        //     optinActive: true,
        //     onlyForNewReg: false,
        //     optinChecked: false
        // },
      ],
      userData: {}, //userdata from RegistratieLight.vue
      // ASSETS:
      logo: 'https://static.nieuwsblad.be/Assets/Images_Upload/2022/10/17/fotoronde_footer.png',
      timerIcon: 'https://static.nieuwsblad.be/Assets/Images_Upload/2022/10/18/fotoronde_timer_icon.png',
      // ARRAYS:
      photosList: [],
      answers: [],

      index: 0,
      currentDay: day,
      playerGuess: '',
      isCorrect: false,
      timerCount: 100,
      similarityCheckResult: 0,
      schifting: '',
      errorSchifting: '',
      // CSS CLASSES:
      correctClass: 'correct',
      falseClass: 'false',
      showReglement: false,
    };
  },
  components: {
    Share,
    RegistratieLight,
    Reglement,
  },
  computed: {
    currentPhoto: function () {
      var self = this;
      return this.photosList.filter(function (show) {
        return show.photoid == self.index;
      });
    },
  },
  methods: {
    registratieIsOK: function () {
      this.currentScreen++;
      this.countDownTimer();
    },
    userDataFromParent: function (value) {
      this.userData = value;
    },
    countDownTimer() {
      if (this.index < 10 && this.timerCount > 0) {
        setTimeout(() => {
          this.timerCount -= 1;
          this.countDownTimer();
        }, 1000);
      }
    },
    getPhotos() {
      fetch('https://fotoronde-default-rtdb.europe-west1.firebasedatabase.app/vragen.json')
        .then((response) => {
          if (response.ok) {
            return response.json();
          }
        })
        .then((data) => {
          const photosList = [];
          for (const id in data) {
            photosList.push({
              id: id,
              photoid: data[id].photoid,
              day: data[id].day,
              photo: data[id].photo,
              answer: data[id].answer,
            });
          }
          console.log(photosList);
          this.photosList = photosList.filter((obj) => {
            return obj.day === this.currentDay;
          });
        })
        .catch((e) => {
          console.log(e);
        });
    },
    checkSimilarity(correctAnswer) {
      var str1 = this.playerGuess.toLowerCase();
      var str2 = correctAnswer.toLowerCase();
      this.similarityCheckResult = this.similarity(str1, str2);
      this.nextPhoto(this.playerGuess);
    },
    similarity(s1, s2) {
      var longer = s1;
      var shorter = s2;
      if (s1.length < s2.length) {
        longer = s2;
        shorter = s1;
      }
      var longerLength = longer.length;
      if (longerLength == 0) {
        return 1.0;
      }
      return (longerLength - this.editDistance(longer, shorter)) / parseFloat(longerLength);
    },
    editDistance(s1, s2) {
      s1 = s1.toLowerCase();
      s2 = s2.toLowerCase();

      var costs = new Array();
      for (var i = 0; i <= s1.length; i++) {
        var lastValue = i;
        for (var j = 0; j <= s2.length; j++) {
          if (i == 0) costs[j] = j;
          else {
            if (j > 0) {
              var newValue = costs[j - 1];
              if (s1.charAt(i - 1) != s2.charAt(j - 1))
                newValue = Math.min(Math.min(newValue, lastValue), costs[j]) + 1;
              costs[j - 1] = lastValue;
              lastValue = newValue;
            }
          }
        }
        if (i > 0) costs[s2.length] = lastValue;
      }
      return costs[s2.length];
    },
    nextPhoto(playerGuess) {
      if (this.similarityCheckResult > 0.69) {
        this.isCorrect = true;
        this.timerCount = this.timerCount + 10;
      } else {
        this.isCorrect = false;
      }
      this.answers.push({ answer: playerGuess, result: this.isCorrect });
      this.playerGuess = '';
      this.index++;
      window.scrollTo(0, 0);
    },
    skipPhoto() {
      this.answers.push({ answer: playerGuess, result: false });
      this.playerGuess = '';
      this.index++;
      window.scrollTo(0, 0);
    },
    saveSchifting() {
      var postToMessagentUrl =
        'https://interactief2.nieuwsblad.be/WedstrijdCR/WedstrijdCR.aspx?ID=Ji6ebWh07E3hyyqE732ubwicUn5UAZOy9srPe1G1csbkDzajh9%2B9X9X7kCJEO_toq5e5BsZR8yVLRT&notags=1&MAIL=';

      var postToMessagentUrlWithParams =
        postToMessagentUrl +
        this.userData.mail +
        '&schifting=' +
        encodeURIComponent(this.schifting) +
        '&tijd=' +
        this.timerCount;

      if (this.schifting != '') {
        this.errorSchifting = '';

        fetch(postToMessagentUrlWithParams)
          .then((response) => response.text())
          .then((response) => {
            response = response.replace(/(\r\n|\n|\r)/gm, '');

            // if (response === 'reedsdeelgenomen') {
            //   this.reedsDeelgenomen = true;
            // } else {
            //   this.reedsDeelgenomen = false;
            // }

            this.currentScreen++;
            //this.getRestoGekozenFromMessagent()
            window.scrollTo(0, 0);
          });
      } else {
        this.errorSchifting = 'Vul de schiftingsvraag in.';
      }
    },
  },
  mounted() {
    this.getPhotos();
  },
};
</script>

<template>
  <RegistratieLight
    v-if="currentScreen == 0"
    @registratieIsOK="registratieIsOK"
    @userDataFromParent="userDataFromParent"
    :optins="optins"
  />
  <div v-if="currentScreen == 1">
    <div v-if="index < 10 && timerCount > 0">
      <div class="timer_digits">
        <span><img class="timer_icon" :src="timerIcon" /></span>
        <span>{{ timerCount }}</span>
      </div>
      <div id="result_bullets">
        <div :class="[answer.result ? correctClass : falseClass]" v-for="answer in answers" :key="answer.result"></div>
      </div>
      <div v-for="vraag in currentPhoto" :key="vraag.id">
        <!-- <div v-if="vraag.day == this.currentDay"> -->
        <h2>{{ vraag.theme }}</h2>
        <img class="foto" :src="vraag.photo" />
        <!-- </div> -->
        <p style="margin: 0px" class="title">Wie of wat zoeken we?</p>
        <p style="margin: 0px" v-if="currentDay == 2 || currentDay == 5 || currentDay == 8" class="title_tip">(Voor- en achternaam)</p>
        <input
          style="margin-top: 10px"
          class="input_form"
          v-on:keyup.enter="checkSimilarity(vraag.answer)"
          id="playerGuess"
          type="search"
          v-model="playerGuess"
          autofocus
        /><br />
        <button @click="checkSimilarity(vraag.answer)">VOLGENDE</button><br />
        <span class="skip" @click="skipPhoto">Ik pas</span>
      </div>
    </div>
    <div v-else-if="index < 10 && timerCount === 0">
      <div class="logo"><img :src="logo" width="320" /></div>
      <h3 class="label_title">Helaas, je tijd is op</h3>
      <p class="intro">Probeer morgen opnieuw.</p>
      <Share /><br />
      <h3 class="label_title">De correcte antwoorden waren:</h3>
      <div v-for="vraag in photosList" :key="vraag.id">
        <p class="correct_answer">{{ vraag.answer }}</p>
      </div>
    </div>
    <div v-else>
      <div id="result_bullets">
        <div :class="[answer.result ? correctClass : falseClass]" v-for="answer in answers" :key="answer.result"></div>
      </div>

      <div>
        <h3 class="label_title">'t Is gebeurd!<br />Je hebt nog {{ timerCount }} seconden over.<br /><br /></h3>

        <p class="intro">
          Wil jij kans maken op een <strong>bol.com cadeaukaart t.w.v. 250 euro</strong>? Vul dan de schiftingsvraag in.
        </p>

        <div>
          <div v-if="errorSchifting != ''">
            <div class="error">
              {{ errorSchifting }}
            </div>
          </div>
          <p class="intro" style="color: #f7d611 !important; font-weight: bold !important">
            Hoeveel personen zullen vandaag deelgenomen hebben aan de wedstrijd om middernacht?
          </p>
          <input
            class="input_form"
            type="number"
            name="SCHIFTING"
            v-model="schifting"
            placeholder="Bvb: 12345"
            autocomplete="false"
            required
          />
          <br />
          <button @click="saveSchifting">VOLGENDE</button><br />
        </div>
      </div>
    </div>
  </div>
  <div v-if="currentScreen == 2">
    <!-- overlay -->
    <div class="overlay" v-if="showReglement" @click="showReglement = false"></div>

    <!-- modal -->
    <div class="modal" v-if="showReglement">
      <div class="close" @click="showReglement = false">x</div>
      <Reglement />
    </div>
    <div class="logo"><img :src="logo" width="320" /></div>
    <p class="intro">
      We hebben je antwoord goed ontvangen. De winnaars worden persoonlijk gecontacteerd. <br /><br />
      <span style="color: #f7d611; font-weight: bold">Kom morgen zeker terug voor een nieuwe fotoronde.</span>
    </p>
    <Share /><br />
    <h3 class="label_title">De correcte antwoorden waren:</h3>
    <div v-for="vraag in photosList" :key="vraag.id">
      <p class="correct_answer">{{ vraag.answer }}</p>
    </div>
    <div class="footer">
      <div @click="showReglement = true">Reglement</div>
    </div>
  </div>

  <!-- <div class="logo"><img :src="logo" width="120" /></div> -->
</template>
