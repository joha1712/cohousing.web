<template>
     <div id="app" class="container">
        <section class="section">           

            <div class="columns is-centered is-mobile" style="margin-bottom: 0">
                <div class="column is-12 has-text-centered">
                    <h1 class="title is-hidden-mobile-bottommarginfix">
                        <span class="control is-pulled-left is-hidden-mobile">
                            <button class="button is-normal" @click="previousweek()" style="background-color:black;">Forrige uge</button>
                        </span>
                        <a @click.prevent="softReload()">{{ pageTitle }}</a>
                        <span class="control is-pulled-right is-hidden-mobile">
                            <button class="button is-normal" @click="nextweek()" style="background-color:black;">Næste uge</button>
                        </span>
                        <span class="control is-pulled-right" style="margin-left:6px;margin-right:6px;">
                            <button class="button is-normal" @click="loadNavigationModal()" style="background-color:black;">
                                <span class="icon is-normal">
                                    <i class="fas fa-lg fa-bars"></i>
                                </span>
                            </button>
                        </span>                        
                    </h1>
                    <p class="subtitle is-hidden-touch">
                        <a @click.prevent="softReload()">{{ pageSubtitle }}</a>
                    </p>
                </div>
            </div>

            <nav class="pagination is-visible-mobile-only" role="navigation" aria-label="pagination">
                <a class="pagination-previous" @click="previousday()">Forrige</a>
                <a class="pagination-next" @click="nextday()">Næste</a>
            </nav>            

        </section>       

        <section class="section" style="padding-top: 0">
            <div class="tile is-ancestor">                
                <div class="meal-tile tile" v-for="(commonMeal, idx2) in commonMeals" :class="calcMealTileClass(idx2)" v-disable-all="!commonMeal.isMealOpen">
                    <div class="tile is-child box" style="padding-bottom:1rem;padding-top:1rem;" >
                        <div class="box has-text-centered meal-header-box" :class="calcMealTitleClass(commonMeal.chefs)" style="margin-bottom: 0" @click="calcMealTitleClass(commonMeal.chefs) === 'meal-chefs-ok' ? showShoppingInfoModal(commonMeal) : false">
                            <p class="title is-4">{{ commonMeal.dayName }} 
                                <span class="icon is-pulled-right" v-if="!commonMeal.isMealOpen">
                                    <i class="fas fa-key"></i>
                                </span>
                            <p class="subtitle is-6">{{ commonMeal.dateName }}</p>                            
                        </div>                        
                        
                        <div class="box meal-chef-box">
                            <div class="meal-chef-row columns is-mobile is-marginless" v-for="(chef, idx) in commonMeal.chefs">
                                <div class="column is-paddingless-top-bottom is-12" >
                                    <div class="field">
                                        <p class="control is-expanded has-icons-left">
                                            <span class="select is-fullwidth">
                                                <select v-model="chef.personId" @change="saveChef(chef)">
                                                    <option :value="null">Vælg {{ idx + 1 }}. kok ...</option>
                                                    <option :value="person.id" v-for="person in people">{{ person.name }}</option>                                            
                                                </select>
                                            </span>
                                            <span class="icon is-small is-left">
                                                <i class="fas fa-user"></i>
                                            </span>                                    
                                        </p>
                                    </div>                                  
                                </div>
                            </div>
                            
                            <div class="meal-chef-row columns is-mobile is-marginless">
                                <div class="column is-paddingless-top-bottom is-12" >
                                    <div class="field has-addons">
                                        <div class="control is-expanded">
                                            <p class="control is-fullwidth">
                                                <input class="input" type="text" v-model="commonMeal.note" @change="saveNote(commonMeal)" placeholder="[Noter]">                                                
                                            </p>
                                        </div>
                                        <div class="control">
                                            <button type="submit" class="button disable-all-skip" :disabled="calcMealTitleClass(commonMeal.chefs) !== 'meal-chefs-ok'" @click="showShoppingInfoModal(commonMeal)">
                                                <span class="icon is-small is-left">
                                                    <i class="fas fa-shopping-basket"></i>
                                                </span>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="box meal-reservation-group" v-for="(group, gIdx) in commonMeal.registrationGroups">
                            <div class="meal-reservation-row columns is-vcentered is-mobile is-marginless" v-for="(registration, index) in group.registrations" :class="calcMealRowClass(registration.regNo)">
                                <div class="column is-5 is-paddingless-top-bottom">
                                    {{ registration.personName }}
                                </div>                            
                                
                                <div class="column is-3 is-paddingless-top-bottom">
                                    <!-- This level is need to keep the radio together in the column (so it doesn't split line when resizing) -->
                                    <nav class="level">                                         
                                        <div class="level-left">
                                            <div class="level-item">
                                                <div class="field">
                                                    <input type="checkbox" :id="registration.id" :name="registration.id" @change="saveRegistration(registration)" class="switch is-rounded is-outlined" v-model="registration.attending" >
                                                    <label :for="registration.id"></label>
                                                </div>
                                            </div>
                                        </div>
                                    </nav>                                
                                </div>

                              <div class="column is-4 is-paddingless-top-bottom">
                                  <p class="control is-pulled-right" style="margin-right:2px;margin-left:2px">
                                    <button class="button is-small" :class="calcGuestBtnClass(registration)" @click="loadGuestsRegModal(registration)">
                                          <span class="icon is-small" >
                                              <i class="fas fa-user-plus"></i>
                                          </span>
                                    </button>
                                  </p>
                                
                                  <p class="control is-pulled-right" style="margin-right:2px;margin-left:2px">
                                    <button class="button is-small" :class="{ 'takeaway-btn-is-active' : registration.takeAway }" @click="saveRegistrationTakeAway(registration)">
                                    <span class="icon is-small" >
                                        <i class="fas fa-shipping-fast"></i>
                                    </span>
                                    </button>
                                  </p>
                              </div>
                            </div>
                        </div>                                                   
                    </div>
                </div>               
            </div>        
        </section>

        <div id="shoppingInfoModal" class="modal" :class="{ 'is-active' : shoppingInfoModalActive }" v-if="shoppingInfo">
            <div class="modal-background"></div>
            <div class="modal-card">
                <header class="modal-card-head">
                    <p class="modal-card-title has-text-centered">{{ shoppingInfo.dayName }} {{ shoppingInfo.dateName }}</p>
                    <button class="delete" aria-label="close" @click="hideShoppingInfoModal()"></button>
                </header>
                <section class="modal-card-body">

                    <div class="columns is-mobile">
                        <div class="column is-3">
                            <label class="label is-pulled-right">Antal</label>                                                        
                        </div>
                        <div class="column is-4">
                            <p class="control has-icons-left is-narrow">
                                <input disabled class="input" type="text" :value="calcShoppingInfoPersonGroupText(shoppingInfo.adults)">
                                <span class="icon is-left">
                                        <i class="fas fa-user"></i>
                                </span>
                            </p>
                        </div>
                        <div class="column is-4">
                            <p class="control has-icons-left is-narrow">
                                <input disabled class="input" type="text" :value="calcShoppingInfoPersonGroupText(shoppingInfo.children)">
                                <span class="icon is-left">
                                    <i class="fas fa-child"></i>
                                </span>
                            </p>                            
                        </div>
                    </div>

                    <div class="columns is-mobile" >
                        <div class="column is-3">
                            <label class="label is-pulled-right">Budget (kr)</label>
                        </div>
                        <div class="column is-4">
                            <p class="control has-icons-left">
                                <input disabled class="input" type="text" :value="shoppingInfo.budget">
                                <span class="icon is-small is-left">
                                    <i class="fas fa-balance-scale"></i>
                                </span>
                            </p>
                        </div>
                    </div>

                    <div class="columns is-mobile" v-for="expense in shoppingInfo.expenses">
                        <div class="column is-3">
                            <label class="label is-pulled-right">Indkøb {{ expense.personName }}</label>
                        </div>
                        <div class="column is-4">
                            <p class="control has-icons-left is-narrow">
                                <input class="input" type="text" v-model="expense.amount" placeholder="[Beløb i kr]" @change="saveExpense(expense)">
                                <span class="icon is-small is-left">
                                    <i class="fas fa-coins"></i>
                                </span>
                            </p>
                        </div>
                    </div>                   

                    <div class="columns is-mobile" v-if="activeMeal">
                        <div class="column is-3">
                            <label class="label is-pulled-right">Tilmelding åben</label>
                        </div>
                        <div class="column is-4">
                            <input type="checkbox" id="open4reg" name="open4reg" class="switch is-rounded is-outlined switch-off-is-red" v-model="activeMeal.isMealOpen" @change="saveOpenStatus(activeMeal)" >
                            <label for="open4reg"></label>
                        </div>
                    </div>                              
                </section>
                <footer class="modal-card-foot">
                    <button class="button is-success" @click="hideShoppingInfoModal()">Luk</button>                    
                </footer>
            </div>
        </div>

        <div id="guestsRegModal" class="modal" :class="{ 'is-active' : guestsRegModalActive }" v-if="guestsReg">
            <div class="modal-background"></div>
            <div class="modal-card">
                <header class="modal-card-head">
                    <p class="modal-card-title has-text-centered">Gæster til {{ guestsReg.personName }} </p>
                    <button class="delete" aria-label="close" @click="hideGuestsRegModal()"></button>
                </header>
                <section class="modal-card-body">                     
                    
                    <div class="field is-horizontal">
                        <div class="field-label">
                            <label class="label">Antal voksne</label>
                        </div>
                        <div class="field-body">
                            <div class="field is-narrow">
                                <p class="control has-icons-left">
                                    <input class="input" type="text" v-model="guestsReg.guests.adults.conventional" @change="saveGuestsReg(guestsReg)" placeholder="[Antal]">
                                    <span class="icon is-left">                                        
                                        <i class="fas fa-drumstick-bite"></i>
                                    </span>
                                </p>
                            </div>
                            <div class="field is-narrow">
                                <p class="control has-icons-left is-narrow">
                                    <input class="input" type="text" v-model="guestsReg.guests.adults.vegetarians" @change="saveGuestsReg(guestsReg)" placeholder="[Antal vegetarer]">
                                    <span class="icon is-left">
                                        <i class="fas fa-seedling"></i>
                                    </span>                                    
                                </p>
                            </div>                            
                        </div>
                    </div>
            
                    <div class="field is-horizontal">
                        <div class="field-label">
                            <label class="label">Antal børn</label>
                        </div>
                        <div class="field-body">
                            <div class="field is-narrow">
                                <p class="control has-icons-left">
                                    <input class="input" type="text" v-model="guestsReg.guests.children.conventional" @change="saveGuestsReg(guestsReg)" placeholder="[Antal]">
                                    <span class="icon is-left">                                        
                                        <i class="fas fa-drumstick-bite"></i>
                                    </span>
                                </p>
                            </div>
                            <div class="field is-narrow">
                                <p class="control has-icons-left is-narrow">
                                    <input class="input" type="text" v-model="guestsReg.guests.children.vegetarians" @change="saveGuestsReg(guestsReg)" placeholder="[Antal vegetarer]">
                                    <span class="icon is-left">                                       
                                        <i class="fas fa-seedling"></i>
                                    </span>                                                                   
                                </p>
                            </div>                            
                        </div>
                    </div>                   
                </section>
                <footer class="modal-card-foot">
                    <button class="button is-success" @click="hideGuestsRegModal()">Luk</button>                    
                </footer>
            </div>
        </div>

        <mainmenu id="navigationModal" :class="{ 'is-active' : navigationModalActive }" v-if="navigationModalActive" @close="hideNavigationModal()" />

        <alertbox id="alertBox" :message="alertBoxMessage" header="Øv - der skete en fejl!" :class="{ 'is-active' : alertBoxActive }" v-if="alertBoxActive" @close="hideAlertBox()" />
            
    </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import mainmenu from "../components/MainMenu.vue"
import alertbox from "../components/AlertBox.vue"
import bulmaSwitch from "bulma-switch";
import {config} from "../scripts/config.js";
import {faUser} from "@fortawesome/free-solid-svg-icons/faUser";
import {faCheck} from "@fortawesome/free-solid-svg-icons/faCheck";
import {faKey} from "@fortawesome/free-solid-svg-icons/faKey";
import {faBars} from "@fortawesome/free-solid-svg-icons/faBars";
import {faShoppingBasket} from "@fortawesome/free-solid-svg-icons/faShoppingBasket";
import {faUserPlus} from "@fortawesome/free-solid-svg-icons/faUserPlus";
import {faChild} from "@fortawesome/free-solid-svg-icons/faChild";
import {faBalanceScale} from "@fortawesome/free-solid-svg-icons/faBalanceScale";
import {faCoins} from "@fortawesome/free-solid-svg-icons/faCoins";
import {faDrumstickBite} from "@fortawesome/free-solid-svg-icons/faDrumstickBite";
import {faSeedling} from "@fortawesome/free-solid-svg-icons/faSeedling";
import {faShippingFast} from "@fortawesome/free-solid-svg-icons/faShippingFast";
import {dom, library} from "@fortawesome/fontawesome-svg-core";

library.add(faUser, faCheck,faKey, faBars, faShoppingBasket, faUserPlus, faChild, faBalanceScale, faCoins, faDrumstickBite, faSeedling, faShippingFast );

    dom.watch();
    
    // AutoUpdate timer (1000 milliseconds = 1 second)
    var inactivityTime = function (timeout, callback) {
        var timerId
        window.addEventListener('load', resetTimer, true);
        var events = ['mousedown', 'mousemove', 'keypress', 'scroll', 'touchstart'];
        events.forEach(function (name) {
            document.addEventListener(name, resetTimer, true)
        });

        function doIdleAction() {
            console.log("doIdleAction");
            timerId = setTimeout(doIdleAction, timeout)
            callback()
        }

        function resetTimer() {
            clearTimeout(timerId)
            timerId = setTimeout(doIdleAction, timeout)
        }
    };

    export default {
        components: {
            mainmenu,
            alertbox
        },
        methods: {
            navigateTo(url) {
                window.location.href = url;
            },

            nextday() {
                if (this.focusMealIdx >= this.commonMeals.length - 1) {
                    this.loadMeals("nextweek", () => {
                        this.focusMealIdx = 0;
                    });
                    return;
                }
                this.loadMeals("week", () => {
                    this.focusMealIdx++;
                });
            },
            previousday() {
                if (this.focusMealIdx <= 0) {
                    this.loadMeals("previousweek", () => {
                        this.focusMealIdx = this.commonMeals.length - 1;
                    });
                    return;
                }
                this.loadMeals("week", () => {
                    this.focusMealIdx--;
                });

            },
            showShoppingInfoModal(meal) {
                axios
                    .get(`${config.baseApiUrl}/commonmeals/shoppinginfo`, {
                        params: {
                            mealId: meal.id
                        }
                    })
                    .then(response => {
                        this.shoppingInfo = response.data;
                        this.activeMeal = meal;
                        this.shoppingInfoModalActive = true;
                    })
                    .catch(e => {
                        alert(`Error fetching shopping info for meal ${meal.id}: ${e}`);
                        console.log(e);
                    });
            },
            hideShoppingInfoModal(meal) {
                this.shoppingInfo = null;
                this.activeMeal = null;
                this.shoppingInfoModalActive = false;
            },
            showAlertBox(message) {
              this.showAlertBoxMessage = message;
              this.alertBoxActive = true;           
            },
            hideAlertBox(message) {              
              this.alertBoxActive = false;
            },
            loadGuestsRegModal(registration) {
                this.guestsRegModalActive = true;
                this.guestsReg = registration;
            },
            hideGuestsRegModal(registration) {
                this.guestsRegModalActive = false;
                this.guestsReg = null;
            },
            loadNavigationModal() {
                this.navigationModalActive = true;                
            },
            hideNavigationModal() {
                this.navigationModalActive = false;
            },
            previousweek() {
                this.loadMeals("previousweek");
            },
            nextweek() {
                this.loadMeals("nextweek");
            },
            calcShoppingInfoPersonGroupText(personGroup) {
                if (personGroup.vegetarians === 0) {
                    return personGroup.total;
                }
                if (personGroup.vegetarians === 1) {
                    return `${personGroup.total} (1 vegetar)`;
                }
                return `${personGroup.total} (${personGroup.vegetarians} vegetarer)`;

            },
            loadMeals(actionName, onSuccess) {
                onSuccess = onSuccess || (_ => {});
                let sortExpr = this.$route.query.s || "";

                axios
                    .get(`${config.baseApiUrl}/commonmeals/list/${actionName}`, {
                        params: {
                            weekDate: this.weekDate,
                            sortExpr: sortExpr
                        }
                    })
                    .then(response => {
                        response.data.meals.forEach(meal => {
                            meal.isMealOpen = meal.status === "OPEN";
                        });
                        this.weekDate = response.data.weekDate;
                        this.commonMeals = response.data.meals;
                        this.people = response.data.people;
                        this.pageSubtitle = response.data.subtitle;
                        this.pageTitle = response.data.title;
                        this.commonMeals[0].enabled = false;
                        onSuccess();
                    })
                    .catch(e => {                        
                        alert(`Error fetching meals for ${actionName}: ${e}`);
                        console.log(e);
                    })

            },
            calcMealRowClass(regNo) {
                return (regNo % 2 === 0) ? "meal-reservation-row-even" : "meal-reservation-row-odd";
            },
            calcMealTileClass(mealIdx) {
                // Only show the active meal on mobile displays
                return mealIdx === this.focusMealIdx ?
                    "" :
                    "is-hidden-mobile";
            },
            calcMealTitleClass(chefs) {
                return chefs.every(x => x.personId != null) ?
                    "meal-chefs-ok" :
                    chefs.some(x => x.personId != null) ? "meal-chefs-missingsome" : "";
            },
            calcChefClass(chef) {
                return chef.personId != null ? ["meal-chef-ok", "fa-check"] : "fa-user";
            },
            calcGuestBtnClass(registration) {
                return registration.guests.adults.conventional > 0 || registration.guests.adults.vegetarians > 0 || registration.guests.children.conventional > 0 || registration.guests.children.vegetarians > 0 ? "guest-btn-has-guests" : "guest-btn-has-guestsno";
            },                     
            saveRegistration(registration, newReg) {              
                // Use newReg if given
                newReg = newReg || registration;

                // Make sure take away is not enabled alone!
                newReg.takeAway = newReg.attending ? newReg.takeAway : false;
                
                axios
                    .put(`${config.baseApiUrl}/commonmeals/registrations`, newReg)
                    .then(response => {                        
                        console.log("Saved registration: " + registration.id);
                        // Copy back changes (because they are ok)
                        Object.keys(newReg).forEach(function (key) {
                            Vue.set(registration, key, newReg[key]);
                          });                        
                    })
                    .catch(e => {   
                        let isFullMeal = e.response && e.response.data && (e.response.data.errorCode === 1100 || e.response.data.errorCode === 1105);
                        this.alertBoxMessage = isFullMeal ? ("Måltidet er fyldt op" + (e.response.data.errorCode === 1100 ? " - måske du har lyst til Take Away?" : " for Take Away")) : e.message;
                        this.alertBoxActive = true;
                        
                        // HACK!!!!
                        if (isFullMeal) {
                          registration.attending = false;
                          registration.takeAway = false;
                        }
                        console.log(e);
                    })
            },
            saveRegistrationTakeAway(registration) {
              let clone = Vue.util.extend({}, registration);
              clone.takeAway = !registration.takeAway; // Toggle takeaway flag
              clone.attending = clone.takeAway ? true : clone.attending; // toggle attending flag (if necessary)
              console.log("clone.attending: " + clone.attending + "  clone.takeAway: " + clone.takeAway);
              this.saveRegistration(registration, clone);              
            },
            saveGuestsReg(reg) {
                this.saveRegistration(reg);
            },
            saveChef(chef) {
                axios
                    .put(`${config.baseApiUrl}/commonmeals/chefs`, chef)
                    .then(response => {
                        console.log("Saved chef: " + chef.id + " personId: " + chef.personId);
                    })
                    .catch(e => {
                        alert("Error saving chef: " + e);
                        console.log(e);
                    })
            },
            saveNote(meal) {
                axios
                    .put(`${config.baseApiUrl}/commonmeals/note`, {
                        id: meal.id,
                        note: meal.note
                    })
                    .then(response => {
                        console.log("Saved note: " + meal.id + " note: " + meal.note);
                    })
                    .catch(e => {
                        alert("Error saving chef: " + e);
                        console.log(e);
                    })
            },
            saveOpenStatus(meal) {
                axios
                    .put(`${config.baseApiUrl}/commonmeals/status`, {
                        id: meal.id,
                        status: meal.isMealOpen ? "OPEN" : "CLOSED"
                    })
                    .then(response => {
                        console.log("Saved meal status: " + meal.id + " status: " + meal.isMealOpen);
                    })
                    .catch(e => {
                        alert("Error saving meal status: " + e);
                        console.log(e);
                    })
            },
            saveExpense(expense) {
                axios
                    .put(`${config.baseApiUrl}/commonmeals/expense`, expense)
                    .then(response => {
                        console.log(`Saved expense ${expense.id} amount: ${expense.amount} for ${expense.personName}`);
                    })
                    .catch(e => {
                        alert("Error saving expense: " + e);
                        console.log(e);
                    })
            },
            softReload() {
                this.loadMeals("activeweek", () => {
                    this.focusMealIdx = Math.max(0, this.commonMeals.findIndex(x => x.isActiveMeal));
                });
            }
        },
        created() {
            this.loadMeals("activeweek", () => {
                this.focusMealIdx = Math.max(0, this.commonMeals.findIndex(x => x.isActiveMeal));                
            });
            let autoupdate = this.$route.query.autoupdate || 600;

            let interval = 1000 * autoupdate;
            inactivityTime(interval, () => {
                this.softReload();
                window.scrollTo(0, 0);
            });
        },        
        data: function () {
            return {
                focusMealIdx: -1,
                pageTitle: "Buske bofællesskab",
                pageSubtitle: "Madplan",
                weekDate: null,
                commonMeals: [],
                people: [],
                shoppingInfoModalActive: false,
                shoppingInfo: {
                    expenses: [],
                    adults: {
                        total: null,
                        vegetarians: null
                    },
                    children: {
                        total: null,
                        vegetarians: null
                    }
                },
                activeMeal: {
                    isMealOpen: true,
                },
                guestsRegModalActive: false,
                guestsReg: null,
                navigationModalActive: false,
                alertBoxActive: false,
                alertBoxMessage: "",
            };
        }
    };
</script>