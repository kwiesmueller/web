<template>
  <div id="gruff" class="container">
    <div class="col-xs-12 claim-content">
      <h2 class="claim-title">{{debate.title}}</h2>
      <p class="claim-desc">{{debate.desc}}</p>
      <ul class="tags" v-for="item in debate.tags" v-bind:key="item.id">
        <li>
          <a class="tag" @click="goTags(item)">{{item.title}}</a>
        </li>
      </ul>
    </div>
    <div class="col-xs-12 claim-arguments">
      <div class="col-xs-6">
        <div class="col-xs-6 col-sm-6 col-md-10 col-lg-10">
          <p class="argument-favor">For</p>
        </div>
        <div class="col-xs-6 col-sm-6 col-md-2 col-lg-2">
          <i class="fa fa-plus-square argument-favor-icon" aria-hidden="true" @click="argumentFavor()"></i>
        </div>
      </div>
      <div class="col-xs-6">
        <div class="col-xs-7 col-sm-6 col-md-10 col-lg-10">
          <p class="argument-against">Against</p>
        </div>
        <div class="col-xs-5 col-sm-6 col-md-2 col-lg-2">
          <i class="fa fa-plus-square argument-against-icon" aria-hidden="true" @click="argumentAgainst()"></i>
        </div>
      </div>
    </div>

    <!-- LIST ARGUMENTS -->
    <div class="col-xs-12 reset-col">
      <div class="col-xs-6 reset-col" style="padding-right: 5px;">
        <!-- CREATE FAVOR -->
        <div class="col-xs-12 space-10 left block-favor outline" v-if="formFavor">
          <div v-if="isError1" class="alert alert-danger" role="alert">
            {{favorError}}
          </div>
          <div class="form-group">
            <select class="form-control input-favor" v-model="argFavor.opt">
              <option value="" disabled>Select an option</option>
              <option value="1">Create argument with new claim</option>
              <option value="2">Create argument with an existing claim</option>
            </select>
          </div>
          <div class="form-group" v-if="argFavor.opt == '2'">
            <v-select
              :debounce="250"
              :on-search="getOptionsClaim"
              :options="optionsClaim"
              placeholder="Search Claim..."
              label="title"
              class="select-back"
              v-model="argFavor.targetClaim"
            >
            </v-select>
          </div>
          <div class="form-group">
            <input type="text" class="form-control input-favor" placeholder="Argument title" v-model="argFavor.title">
          </div>
          <div class="form-group">
            <textarea type="text" class="form-control input-favor" placeholder="Argument description" v-model="argFavor.desc" rows="4"></textarea>
          </div>
          <v-btn @click="saveFavor()" outline color="primary" slot="activator" class="blue btn-send-favor-it" :disabled="argFavor.title == undefined || argFavor.title == null || argFavor.title == ''">
            Send it
          </v-btn>
          <v-btn @click="cancel()" outline color="primary" slot="activator" class="blue btn-cancel-it">
            Cancel it
          </v-btn>
        </div>

        <!-- LIST ARGUMENTS FAVOR -->
        <div class="col-xs-12 reset-col" v-for="item in debate.pro" v-bind:key="item.id">
          <h2 v-if="item.args.length > 0" style="font-weight: 300; text-transform: uppercase;">{{item.name}}</h2>

          <!-- todo: @click="goClaim(children)" -->
          <div v-for="children in item.args" v-bind:key="children.uuid" class="block-shadow space-10">
            <div class="block-favor block-favor-list" @click="openTab(children, false)">
              <label style="color: #41cc90;">Strength:
                <span style="color: #333; font-size: 12px;">{{children.strength}}</span>
              </label>
              <span> | </span>
              <label style="color: #41cc90;">Truth:
                <span style="color: #333; font-size: 12px;">{{children.claim.truth}}</span>
              </label>
              <h4 style="font-weight: 400;" v-if="children.title != ''">{{children.title}}</h4>
              <h4 style="font-weight: 400;" v-else>{{children.claim.title}}</h4>
              <span>{{children.desc}}</span>
            </div>
            <v-tabs grow icons v-if="children.tab">
              <v-tabs-bar class="grey lighten-4">
                <v-tabs-slider class="tab-slider-favor"></v-tabs-slider>
                <v-tabs-item href="#tab-1">
                  <v-icon>gavel</v-icon>
                  <span class="hidden-sm-and-down">Arguments</span>
                </v-tabs-item>
                <v-tabs-item href="#tab-2">
                  <v-icon>library_books</v-icon>
                  <span class="hidden-sm-and-down">Contexts</span>
                </v-tabs-item>
                <v-tabs-item href="#tab-3">
                  <v-icon>create</v-icon>
                  <span class="hidden-sm-and-down">New</span>
                </v-tabs-item>
                <v-tabs-item @click="goClaim(children)">
                  <v-icon>call_missed_outgoing</v-icon>
                  <span class="hidden-sm-and-down">See Claim</span>
                </v-tabs-item>
              </v-tabs-bar>
              <v-tabs-items class="tab-heigth">
                <v-tabs-content :key="1" :id="'tab-1'">
                  <div v-if="children.args != null">
                    <div class="col-xs-6" style="padding-top: 8px;">
                      <h5>Arguments For</h5>
                      <div class="col-xs-12 reset-col" style="border-bottom: 1px solid #eee;"
                        v-for="arPro in children.args.pro" v-bind:key="arPro.uuid">
                        <label style="color: #41cc90; font-size: 11px; margin-top: 8px; margin-bottom:0;">Strength:
                          <span style="color: #333; font-size: 12px;">{{arPro.strength}}</span>
                        </label>
                        <span> | </span>
                        <label style="color: #41cc90; font-size: 11px;">Truth:
                          <span style="color: #333; font-size: 12px;">{{arPro.claim.truth}}</span>
                        </label>
                        <p style="margin-bottom: 5px;">{{arPro.title}}</p>
                      </div>
                    </div>
                    <div class="col-xs-6" style="padding-top: 8px;">
                      <h5>Arguments Against</h5>
                      <div class="col-xs-12 reset-col" style="border-bottom: 1px solid #eee;"
                        v-for="arCon in children.args.con" v-bind:key="arCon.uuid">
                        <label style="color: #ff725c; font-size: 11px; margin-top: 8px; margin-bottom:0;">Strength:
                          <span style="color: #333; font-size: 12px;">{{arCon.strength}}</span>
                        </label>
                        <span> | </span>
                        <label style="color: #ff725c; font-size: 11px;">Truth:
                          <span style="color: #333; font-size: 12px;">{{arCon.claim.truth}}</span>
                        </label>
                        <p style="margin-bottom: 5px;">{{arCon.title}}</p>
                      </div>
                    </div>
                  </div>
                  <div class="col-xs-12" v-else>
                    <h3 class="text-xs-center" style="margin-top: 37px;">This argument has no Argument</h3>
                  </div>
                </v-tabs-content>
                <v-tabs-content :key="2" :id="'tab-2'">
                  <div class="col-xs-12" style="border-top: 1px solid #ccc; padding-top: 8px;" flat v-for="context in children.claim.contexts" v-bind:key="context.id">
                    <a v-bind:href="context.url" target="_blank">{{context.title}}</a>
                    <p>{{context.desc}}</p>
                  </div>
                  <div class="col-xs-12" v-if="children.claim.contexts == undefined">
                    <h3 class="text-xs-center" style="margin-top: 37px;">This argument has no Context</h3>
                  </div>
                </v-tabs-content>
                <v-tabs-content :key="3" :id="'tab-3'">
                  <div class="col-xs-12" style="padding-top: 10px;">
                    <div class="col-xs-12" style="padding-top: 10px;">
                      <div v-if="isErrorArgumentPro" class="alert alert-danger" role="alert">
                        {{agrProError}}
                      </div>
                      <h4>Create a new Argument</h4>
                      <div class="form-group">
                        <select class="form-control input-favor" v-model="argPro.type">
                          <option value="" disabled>Select an option</option>
                          <option value="1">For</option>
                          <option value="2">Against</option>
                        </select>
                      </div>
                      <div class="form-group">
                        <input type="text" class="form-control input-favor" placeholder="Argument title" v-model="argPro.title">
                      </div>
                      <div class="form-group">
                        <textarea type="text" class="form-control input-favor" placeholder="Argument description" v-model="argPro.desc" rows="4"></textarea>
                      </div>
                      <v-btn @click="createArgumentPro(children)" outline color="primary" slot="activator" class="blue btn-send-favor-it" :disabled="argPro.title == undefined || argPro.title == null || argPro.title == ''">
                        Send it
                      </v-btn>
                    </div>
                  </div>
                </v-tabs-content>
              </v-tabs-items>
            </v-tabs>
          </div>
        </div>
      </div>
      <div class="col-xs-6 reset-col" style="padding-left: 5px;">
        <!-- CREATE AGAINST -->
        <div class="col-xs-12 space-10 left block-against outline" v-if="formAgainst">
          <div v-if="isError2" class="alert alert-danger" role="alert">
            {{againstError}}
          </div>
          <div class="form-group">
            <select class="form-control input-favor" v-model="argAgainst.opt">
              <option value="" disabled>Select an option</option>
              <option value="1">Create argument with new claim</option>
              <option value="2">Create argument with an existing claim</option>
            </select>
          </div>
          <div class="form-group" v-if="argAgainst.opt == '2'">
            <v-select
              :debounce="250"
              :on-search="getOptionsClaim"
              :options="optionsClaim"
              placeholder="Search Claim..."
              label="title"
              class="select-back"
              v-model="argAgainst.targetClaim"
            >
            </v-select>
          </div>
          <div class="form-group">
            <input type="text" class="form-control input-against" placeholder="Argument title" v-model="argAgainst.title">
          </div>
          <div class="form-group">
            <textarea type="text" class="form-control input-against" placeholder="Argument description" v-model="argAgainst.desc" rows="4"></textarea>
          </div>
          <v-btn @click="saveAgainst()" outline color="primary" slot="activator" class="blue btn-send-against-it" :disabled="argAgainst.title == undefined || argAgainst.title == null || argAgainst.title == ''">
            Send it
          </v-btn>
          <v-btn @click="cancel()" outline color="primary" slot="activator" class="blue btn-cancel-it">
            Cancel it
          </v-btn>
        </div>

        <!-- LIST ARGUMENTS AGAINST -->
        <div class="col-xs-12 reset-col" v-for="item in debate.con" v-bind:key="item.id">
          <h2 v-if="item.args.length > 0" style="font-weight: 300; text-transform: uppercase;">{{item.name}}</h2>

          <div v-for="children in item.args" v-bind:key="children.uuid" class="block-shadow space-10">
            <div class="block-against block-against-list" @click="openTab(children, false)">
              <label style="color: #ff725c;">Strength:
                <span style="color: #333; font-size: 12px;">{{children.strength}}</span>
              </label>
              <span> | </span>
              <label style="color: #ff725c;">Truth:
                <span style="color: #333; font-size: 12px;">{{children.claim.truth}}</span>
              </label>
              <h4 style="font-weight: 400;" v-if="children.title != ''">{{children.title}}</h4>
              <h4 style="font-weight: 400;" v-else>{{children.claim.title}}</h4>
              <span>{{children.desc}}</span>
            </div>
            <v-tabs grow icons v-if="children.tab">
              <v-tabs-bar class="grey lighten-4">
                <v-tabs-slider class="tab-slider-against"></v-tabs-slider>
                <v-tabs-item href="#tab-1">
                  <v-icon>gavel</v-icon>
                  <span class="hidden-sm-and-down">Arguments</span>
                </v-tabs-item>
                <v-tabs-item href="#tab-2">
                  <v-icon>library_books</v-icon>
                  <span class="hidden-sm-and-down">Contexts</span>
                </v-tabs-item>
                <v-tabs-item href="#tab-3">
                  <v-icon>create</v-icon>
                  <span class="hidden-sm-and-down">New</span>
                </v-tabs-item>
                <v-tabs-item @click="goClaim(children)">
                  <v-icon>call_missed_outgoing</v-icon>
                  <span class="hidden-sm-and-down">See Claim</span>
                </v-tabs-item>
              </v-tabs-bar>
              <v-tabs-items class="tab-heigth">
                <v-tabs-content :key="1" :id="'tab-1'">
                  <div v-if="children.args != null">
                    <div class="col-xs-6" style="padding-top: 8px;">
                      <h5>Arguments For</h5>
                      <div class="col-xs-12 reset-col" style="border-bottom: 1px solid #eee;"
                        v-for="arPro in children.args.pro" v-bind:key="arPro.uuid">
                        <label style="color: #41cc90; font-size: 11px; margin-top: 8px; margin-bottom:0;">Strength:
                          <span style="color: #333; font-size: 12px;">{{arPro.strength}}</span>
                        </label>
                        <span> | </span>
                        <label style="color: #41cc90; font-size: 11px;">Truth:
                          <span style="color: #333; font-size: 12px;">{{arPro.claim.truth}}</span>
                        </label>
                        <p style="margin-bottom: 5px;">{{arPro.title}}</p>
                      </div>
                    </div>
                    <div class="col-xs-6" style="padding-top: 8px;">
                      <h5>Arguments Against</h5>
                      <div class="col-xs-12 reset-col" style="border-bottom: 1px solid #eee;"
                        v-for="arCon in children.args.con" v-bind:key="arCon.uuid">
                        <label style="color: #ff725c; font-size: 11px; margin-top: 8px; margin-bottom:0;">Strength:
                          <span style="color: #333; font-size: 12px;">{{arCon.strength}}</span>
                        </label>
                        <span> | </span>
                        <label style="color: #ff725c; font-size: 11px;">Truth:
                          <span style="color: #333; font-size: 12px;">{{arCon.claim.truth}}</span>
                        </label>
                        <p style="margin-bottom: 5px;">{{arCon.title}}</p>
                      </div>
                    </div>
                  </div>
                  <div class="col-xs-12" v-else>
                    <h3 class="text-xs-center" style="margin-top: 37px;">This argument has no Argument</h3>
                  </div>
                </v-tabs-content>
                <v-tabs-content :key="2" :id="'tab-2'">
                  <div class="col-xs-12" style="border-top: 1px solid #ccc; padding-top: 8px;" flat v-for="context in children.claim.contexts" v-bind:key="context.id">
                    <a v-bind:href="context.url" target="_blank">{{context.title}}</a>
                    <p>{{context.desc}}</p>
                  </div>
                  <div class="col-xs-12" v-if="children.claim.contexts == undefined">
                    <h3 class="text-xs-center" style="margin-top: 37px;">This argument has no Context</h3>
                  </div>
                </v-tabs-content>
                <v-tabs-content :key="3" :id="'tab-3'">
                  <div class="col-xs-12" style="padding-top: 10px;">
                    <div v-if="isErrorArgumentCon" class="alert alert-danger" role="alert">
                      {{agrConError}}
                    </div>
                    <h4>Create a new Argument</h4>
                    <div class="form-group">
                      <select class="form-control input-against" v-model="argCon.type">
                        <option value="" disabled>Select an option</option>
                        <option value="1">For</option>
                        <option value="2">Against</option>
                      </select>
                    </div>
                    <div class="form-group">
                      <input type="text" class="form-control input-against" placeholder="Argument title" v-model="argCon.title">
                    </div>
                    <div class="form-group">
                      <textarea type="text" class="form-control input-against" placeholder="Argument description" v-model="argCon.desc" rows="4"></textarea>
                    </div>
                    <v-btn @click="createArgumentCon(children)" outline color="primary" slot="activator" class="blue btn-send-against-it" :disabled="argCon.title == undefined || argCon.title == null || argCon.title == ''">
                      Send it
                    </v-btn>
                  </div>
                </v-tabs-content>
              </v-tabs-items>
            </v-tabs>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import auth from '../auth';
import router from '../router';

/* eslint-disable no-undef, no-param-reassign */
const API_URL = API;

export default {
  data() {
    return {
      debate: {},
      formFavor: false,
      formAgainst: false,
      isError1: false,
      isError2: false,
      argFavor: {
        opt: '1',
      },
      argAgainst: {
        opt: '1',
      },
      argPro: {},
      argCon: {},
      isErrorArgumentCon: false,
      isErrorArgumentPro: false,
      userIdLogged: 0,
      optionsClaim: [],
    };
  },

  created() {
    this.userIdLogged = auth.getLoggedId();
    this.list();
  },

  watch: {
    $route: 'list',
  },

  methods: {
    groupBy(args) {
      const newArguments = [];
      const argsVeryWeak = [];
      const argsWeak = [];
      const argsModerate = [];
      const argsStrong = [];
      const argsVeryStrong = [];
      for (let i = 0; i < args.length; i += 1) {
        args[i].tab = false;
        if (args[i].strength >= 0 && args[i].strength < 1) {
          argsVeryWeak.push(args[i]);
        } else if (args[i].strength >= 1 && args[i].strength < 2) {
          argsWeak.push(args[i]);
        } else if (args[i].strength >= 2 && args[i].strength < 3) {
          argsModerate.push(args[i]);
        } else if (args[i].strength >= 3 && args[i].strength < 4) {
          argsStrong.push(args[i]);
        } else if (args[i].strength >= 4 && args[i].strength < 5) {
          argsVeryStrong.push(args[i]);
        }
      }

      newArguments.push(
        {
          id: 1,
          name: 'Very Strong',
          args: argsVeryStrong,
        },
        {
          id: 2,
          name: 'Strong',
          args: argsStrong,
        },
        {
          id: 3,
          name: 'Moderate',
          args: argsModerate,
        },
        {
          id: 4,
          name: 'Weak',
          args: argsWeak,
        },
        {
          id: 5,
          name: 'Very Weak',
          args: argsVeryWeak,
        },
      );

      return newArguments;
    },

    list() {
      axios.get(`${API_URL}/claims/${this.$route.params.id}`).then((response) => {
        const debate = response.data;
        if (debate.pro !== undefined) {
          for (let i = 0; i < debate.pro.length; i += 1) {
            debate.pro[i].isShow = false;
          }
          debate.pro = this.groupBy(debate.pro);
        }

        if (debate.con !== undefined) {
          for (let i = 0; i < debate.con.length; i += 1) {
            debate.con[i].isShow = false;
          }
          debate.con = this.groupBy(debate.con);
        }

        this.debate = debate;
      });
    },

    getOptionsClaim() {
      axios.get(`${API_URL}/claims`).then((response) => {
        this.optionsClaim = response.data;
      });
    },

    argumentFavor() {
      this.argFavor = {
        opt: '1',
      };
      this.formAgainst = false;
      this.formFavor = !this.formFavor;
    },

    argumentAgainst() {
      this.argAgainst = {
        opt: '1',
      };
      this.formFavor = false;
      this.formAgainst = !this.formAgainst;
    },

    edit(type, item) {
      if (type === 'favor') {
        this.argumentFavor();
        this.argFavor.uuid = item.uuid;
        if (item.title !== '') {
          this.argFavor.title = item.title;
        } else {
          this.argFavor.title = item.claim.title;
        }

        if (item.desc !== '') {
          this.argFavor.desc = item.desc;
        } else {
          this.argFavor.desc = item.claim.desc;
        }
      } else {
        this.argumentAgainst();
        this.argAgainst.uuid = item.uuid;
        if (item.title !== '') {
          this.argAgainst.title = item.title;
        } else {
          this.argAgainst.title = item.claim.title;
        }

        if (item.desc !== '') {
          this.argAgainst.desc = item.desc;
        } else {
          this.argAgainst.desc = item.claim.desc;
        }
      }
    },

    saveFavor() {
      const model = {
        targetClaimID: this.$route.params.id,
        type: 1,
        claim: {
          title: this.argFavor.title,
          desc: this.argFavor.desc,
        },
        title: this.argFavor.title,
        desc: this.argFavor.desc,
      };

      if (this.argFavor.targetClaim !== undefined) {
        model.claimId = this.argFavor.targetClaim.uuid;
        model.claim.uuid = this.argFavor.targetClaim.uuid;
      }

      if (this.argFavor.uuid === undefined) {
        axios.post(`${API_URL}/arguments`, model).then(() => {
          this.formFavor = false;
          this.argFavor = {};
          this.list();
        }, () => {
          this.isError1 = true;
          this.favorError = 'You must be logged.';
          setTimeout(() => {
            this.isError1 = false;
          }, 2000);
        });
      } else {
        const modelUpdate = {
          title: this.argFavor.title,
          desc: this.argFavor.desc,
        };
        axios.put(`${API_URL}/arguments/${this.argFavor.uuid}`, modelUpdate).then(() => {
          this.formFavor = false;
          this.argFavor = {};
          this.list();
        }, () => {
          this.isError1 = true;
          this.favorError = 'You must be logged.';
          setTimeout(() => {
            this.isError1 = false;
          }, 2000);
        });
      }
    },

    saveAgainst() {
      const model = {
        targetClaimID: this.$route.params.id,
        type: 2,
        claim: {
          title: this.argAgainst.title,
          desc: this.argAgainst.desc,
        },
        title: this.argAgainst.title,
        desc: this.argAgainst.desc,
      };

      if (this.argAgainst.targetClaim !== undefined) {
        model.claimId = this.argAgainst.targetClaim.uuid;
        model.claim.uuid = this.argAgainst.targetClaim.uuid;
      }

      if (this.argAgainst.uuid === undefined) {
        axios.post(`${API_URL}/arguments`, model).then(() => {
          this.formAgainst = false;
          this.argAgainst = {};
          this.list();
        }, () => {
          this.isError2 = true;
          this.againstError = 'You must be logged.';

          setTimeout(() => {
            this.isError2 = false;
          }, 2000);
        });
      } else {
        const modelUpdate = {
          title: this.argAgainst.title,
          desc: this.argAgainst.desc,
        };

        axios.put(`${API_URL}/arguments/${this.argAgainst.uuid}`, modelUpdate).then(() => {
          this.formAgainst = false;
          this.argAgainst = {};
          this.list();
        }, () => {
          this.isError2 = true;
          this.againstError = 'You must be logged.';

          setTimeout(() => {
            this.isError2 = false;
          }, 2000);
        });
      }
    },

    createArgumentPro(item) {
      const model = {
        targetArgID: item.uuid,
        type: parseInt(this.argPro.type, 10),
        title: this.argPro.title,
        desc: this.argPro.desc,
      };

      axios.post(`${API_URL}/arguments`, model).then(() => {
        this.argPro = {};
        this.loadArgs(item.uuid, (result) => {
          item.args = result;
          // hack to force update
          item.tab = !item.tab;
          item.tab = !item.tab;
        });
      }, () => {
        this.isErrorArgumentPro = true;
        this.agrProError = 'You must be logged.';
        setTimeout(() => {
          this.isErrorArgumentPro = false;
        }, 2000);
      });
    },

    createArgumentCon(item) {
      const model = {
        targetArgID: item.uuid,
        type: parseInt(this.argCon.type, 10),
        title: this.argCon.title,
        desc: this.argCon.desc,
      };

      axios.post(`${API_URL}/arguments`, model).then(() => {
        this.argCon = {};
        this.loadArgs(item.uuid, (result) => {
          item.args = result;
          // hack to force update
          item.tab = !item.tab;
          item.tab = !item.tab;
        });
      }, () => {
        this.isErrorArgumentCon = true;
        this.agrConError = 'You must be logged.';
        setTimeout(() => {
          this.isErrorArgumentCon = false;
        }, 2000);
      });
    },

    openTab(item, reload) {
      if (item.tab && !reload) {
        item.tab = !item.tab;
        return;
      }

      this.loadArgs(item.uuid, (result) => {
        item.args = result;

        if (!reload) item.tab = !item.tab;
      });
    },

    loadArgs(uuid, cb) {
      axios.get(`${API_URL}/arguments/${uuid}`).then((response) => {
        let argument = response.data;
        if (typeof argument.pro === 'undefined') {
          argument.pro = [];
        }
        if (typeof argument.con === 'undefined') {
          argument.con = [];
        }


        if (argument.pro.length === 0 && argument.con.length === 0) {
          argument = null;
        }

        cb(argument);
      });
    },

    cancel() {
      this.formAgainst = false;
      this.formFavor = false;
    },

    goClaim(item) {
      router.push(`/gruff/${item.claimId}`);
    },

    goToContext() {
      router.push('/context');
    },

    goTags(item) {
      router.push(`/tags/${item.id}`);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #gruff ul {
    padding:0;
  }
  #gruff a {
    cursor: pointer;
  }

  .claim-content {
    background-color: #fff;
    height: auto;
    border: 1px solid #e6e9eb;
    cursor: pointer;
  }

  .claim-content:hover {
    /* border: 0.5px solid #999999; */
  }

  .claim-title {
    white-space: pre-wrap;
    word-wrap: break-word;
    font-weight: 400;
    color: #3d3d3d;
    margin-top: 22px;
  }

  .claim-desc {
    white-space: pre-wrap;
    word-wrap: break-word;
    font-weight: 400;
    color: #3d3d3d;
  }

  .claim-arguments {
    height: 65px;
    background-color: #fff;
    box-shadow: 0 0px 0px 0px rgba(0,0,0,.2), 0 0px 0px 0 rgba(0,0,0,.14), 0 1px 1px 0 rgba(0,0,0,.12);
  }

  .claim-arguments > div {
    padding: 16px 16px 1px 24px;
  }

  .claim-arguments > div:first-child {
    border-right: 1px solid #e6e9eb;
  }

  .argument-favor {
    font-weight: 700;
    font-size: 21px;
    line-height: 1.5;
    -webkit-box-flex: 1;
    flex-grow: 1;
    cursor: default;
    color: #41cc90;
  }

  .argument-favor-icon {
    font-size: 38px;
    color: #41cc90;
    cursor: pointer;
  }

  .argument-favor-icon:hover {
    color: #2eac76;
  }

  .argument-against {
    font-weight: 700;
    font-size: 21px;
    line-height: 1.5;
    -webkit-box-flex: 1;
    flex-grow: 1;
    cursor: default;
    color: #ff725c;
  }

  .argument-against-icon {
    font-size: 38px;
    color: #ff725c;
    cursor: pointer;
  }

  .argument-against-icon:hover {
    color: #ff4629;
  }

  .left {
    text-align: left;
  }

  .space-10 {
    margin-top: 10px;
  }

  .space-15 {
    margin-top: 15px;
  }

  .space-20 {
    margin-top: 20px;
  }

  .space {
    margin-top: 40px;
  }

  .block-shadow {
    background-color: #fff;
    box-shadow: 0 1px 1px -3px rgba(0,0,0,.2), 0 1px 1px 0 rgba(0,0,0,.14), 0 1px 1px 0 rgba(0,0,0,.12);
    transition: width .2s linear,margin-left .2s linear;
  }

  .block-favor {
    background-color: #fff;
    padding: 20px;
    box-shadow: 0 1px 1px -3px rgba(0,0,0,.2), 0 1px 1px 0 rgba(0,0,0,.14), 0 1px 1px 0 rgba(0,0,0,.12);
    transition: width .2s linear,margin-left .2s linear;
  }

  .block-favor.outline {
    border: 1px solid #41cc90;
  }

  .block-favor-list:hover {
    border: 1px solid #41cc90;
    padding: 19px;
    cursor: pointer;
  }

  .block-against {
    background-color: #fff;
    padding: 20px;
    box-shadow: 0 1px 1px -3px rgba(0,0,0,.2), 0 1px 1px 0 rgba(0,0,0,.14), 0 1px 1px 0 rgba(0,0,0,.12);
    transition: width .2s linear,margin-left .2s linear;
  }

  .block-against.outline {
    border: 1px solid #ff725c;
  }

  .block-against-list:hover {
    border: 1px solid #ff725c;
    padding: 19px;
    cursor: pointer;
  }

  input, textarea {
    border: 1px solid #e6e9eb;
    border-radius: 4px;
    box-shadow: none;
  }
  .input-favor:focus {
    border-color: #e6e9eb;
    box-shadow: none;
  }
  .input-against:focus {
    border-color: #e6e9eb;
    box-shadow: none;
  }

  .btn-send-favor-it {
    border-color: #41cc90 !important;
    border-radius: 4px;
    color: #41cc90;
    font-weight: 800;
  }

  .btn-send-against-it {
    border-color: #ff725c !important;
    border-radius: 4px;
    color: #ff725c;
    font-weight: 800;
  }

  .btn-cancel-it {
    border-color: #a9b0b8 !important;
    border-radius: 4px;
    color: #a9b0b8;
    font-weight: 800;
  }

  @media (max-width: 768px) {
    .claim-arguments > div {
      padding: 16px 0px 1px 0px;
    }
  }

  .tab-slider-favor {
    background-color: #41cc90 !important;
    border-color: #41cc90 !important;
  }

  .tab-slider-against {
    background-color: #ff725c !important;
    border-color: #ff725c !important;
  }

  .tab-heigth {
    max-height: 330px;
    min-height: 100px;
    overflow-y: auto;
  }
</style>
