<template>
    <Panel>
        <ExitCross @click="$emit('emit-close-pop-up')" />
        <ChampForm v-for="(champ, i) in info" :key="champ" :type="champ.type" :name="champ.name" :id="champ.id" :placeholder="champ.placeholder" :required="champ.required" v-model="info[i].modelValue"/>
        <p>Date de naissance :</p>
        <BirthdayForm v-model="birthdate"/>
        <p>Genre :</p>
        <GenderForm @select-option="pickGender"/>
        <BigButton :type="signupButton.type" :class="signupButton.classe" :text="signupButton.text" @click="signupNewUser()"/>
        <p v-show="alert">{{ alertMessage }}</p>
    </Panel>
</template>

<script>
import BirthdayForm from '@/components/IdentityForm/BirthdayForm.vue'
import ChampForm from "@/components/IdentityForm/ChampForm.vue"
import GenderForm from "@/components/IdentityForm/GenderForm.vue"
import ExitCross from "@/components/Buttons/ExitCross.vue"
import BigButton from "@/components/Buttons/BigButton.vue"
import Panel from "@/components/Frames/Panel.vue"
import ApiUserRoutes from "@/services/ApiUserRoutes"
export default {
    name: 'SignupMenu',
    methods: {
        emits: ['emit-close-pop-up'],
        pickGender(payload) {
            this.gender = payload.pickedOption;
        },
        // Demande l'envoi au serveur de la requête HTTP contenant les infos de l'utilisateur à créer et attend sa réponse
        async signupNewUser() {
            const data = {
                firstname: this.info[0].modelValue,
                lastname: this.info[1].modelValue,
                email: this.info[2].modelValue,
                password: this.info[3].modelValue,
                passwordVerif: this.info[4].modelValue,
                birthdate: this.birthdate,
                gender: this.gender
            };

            // Vérifie que le mdp et la confirmation de mdp correspondent
            if (data.password === data.passwordVerif) {
                this.alert = false;
                const isSignedUp = await ApiUserRoutes.signupNewUser(data);
                console.log(isSignedUp);
                // Si la réponse du serveur est la confirmation d'inscription de l'utilisateur, login celui-ci automatiquement
                if (isSignedUp.data.message === "user added") {
                    const loginData = {
                        email: this.info[2].modelValue,
                        password: this.info[3].modelValue,
                    };
                    const isLoggedIn = await ApiUserRoutes.loginUser(loginData);
                    if (isLoggedIn.data.userId && isLoggedIn.data.token) {
                    console.log(isLoggedIn);
                    localStorage.setItem('userId', JSON.stringify(isLoggedIn.data.userId));
                    localStorage.setItem('token', JSON.stringify(isLoggedIn.data.token));
                    this.$router.push({ name: 'Fil' });
                } else {
                        this.alert = true;
                        this.alertMessage = "Erreur de login"
                    }
                } else {
                    this.alert = true;
                    this.alertMessage = "Erreur du serveur, réessayez plus tard."
                }
            } else {
                this.alert = true;
                this.alertMessage = "Les mots de passe de correspondent pas !"
            }
        }
    },
    data() {
        return {
            info: [
                {type: "text", name: "prenom", id: "prenom", placeholder: "Prénom", required: true, modelValue: ""},
                {type: "text", name: "nom", id: "nom", placeholder: "Nom", required: true, modelValue: ""},
                {type: "email", name: "email", id: "emailSignup", placeholder: "Adresse e-mail", required: true, modelValue: ""},
                {type: "password", name: "password", id: "passwordSignup", placeholder: "Mot de passe", required: true, modelValue: ""},
                {type: "password", name: "passwordVerif", id: "passwordSignupVerif", placeholder: "Ré-entrez le mot de passe", required: true, modelValue: ""}
            ],
            signupButton: {type: "submit", classe: "signup", text: "Je m'inscris !"},
            birthdate: "",
            gender: "",
            alert: false,
            alertMessage: ""
        }
    },
    components: {
        BirthdayForm,
        ChampForm,
        GenderForm,
        ExitCross,
        BigButton,
        Panel
    }
}
</script>