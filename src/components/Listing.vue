<template>
  <div>

    <!-- navbar -->
    <nav class="navbar" role="navigation" aria-label="main navigation">
        <div class="navbar-item">
            <p>Coding assingment: Sortable list of representatives</p>
        </div>
        <div class="navbar-end">
            <div class="navbar-item">
                <p>Made with vue</p>
            </div>
        </div>
    </nav>        

    <!-- content -->
    <section class="section">
        <div class="container has-text-centered">
          <h3 class="title">Sortable list of representatives</h3>
          <!-- sorting controls -->
          <div class="columns is-centered">
            <div class="column is-3">
                <!-- sort by -->
                <div class="field is-fullwidth">
                    <label>Sort by</label>
                    <div class="select is-fullwidth">
                        <select v-model="selectedSorting" 
                        v-on:change="sortListing($event, $event.target.value)" v-cloak>                        
                            <option :value="-1">Select</option>
                            <option v-for="s in sortingOptions"                                       
                                    :key="s.value"
                                    :value="s.value"
                            >{{ s.text }}
                            </option>
                        </select>
                    </div>
                </div>
            </div>
            <div class="column is-3">
                <!-- display by party - only major parties -->
                <div class="field is-fullwidth">
                    <label>Display by party</label>
                    <div class="select  is-fullwidth">
                        <select @change="selectedParty = $event.target.value" v-cloak>
                            <option :value="-1">All</option>
                            <option v-for="p in parties" 
                                    :key="p.id"
                                    :value="p.id"
                            >{{ p.name }}
                            </option>
                        </select>
                    </div>
                </div>
            </div> 
          </div> <!-- columns -->
        </div>
        
        <!-- listing -->
        <div id="listing" class="container has-text-centered">          
          <div class="columns is-centered">
            <div class="column is-7" style="overflow:hidden">
                <!-- loader -->
                <div class="loader-wrapper">
                    <div class="loader is-loading"></div>
                    &nbsp; Please wait... loading representatives
                </div>

                <table class="table" v-cloak>
                    <thead>
                        <th></th>
                        <th>Namn
                          <span class="birthYearTableHeader">Född</span>
                        </th>
                        <th>Parti</th>
                        <th>Valkrets</th>
                        <th>Votering<br><small class="voteNumsLabel">Ja Nej Avstår</small></th>
                    </thead>
                    <tbody>
                        <tr v-for="rep in reps" :key="rep.hangar_id" v-show="selectedParty != '-1' ? rep.parti == selectedParty : true">
                            <td><img :src="rep.bild_url_80"></td>
                            <td>{{ rep.sorteringsnamn | displayFullname }}
                                <span class="age">{{ rep.fodd_ar  }}</span>
                                <br><small>{{ rep.status }}</small>
                            </td>
                            <td class="party">{{ rep.parti }}</td>
                            <td>{{ rep.valkrets }}</td>
                            <td class="voteNumWrapper">
                                <span class="voteNum ja">{{ getRepVotes(rep.intressent_id, 'Ja')  }}</span> 
                                <span class="voteNum nej">{{ getRepVotes(rep.intressent_id, 'Nej') }}</span> 
                                <span class="voteNum avs">{{ getRepVotes(rep.intressent_id, 'Avs') }}</span>
                            </td>                                
                            <!-- '<span class="is-success">' + votesJa + '</span><span class="is-danger">' + votesNej + '</span><span class="is-warning">' + votesAvstar + '</span>';
                            -->
                        </tr>
                    </tbody>
                </table>
            </div> 
          </div> <!-- columns -->

        </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Listing3',
  data() {
      return {
          sortingOptions: [ 
              { "text": "Född år", "value": "fodd_ar" }, 
              { "text": "Kön", "value": "kon" }, 
              { "text": "Efternamn", "value": "sorteringsnamn" }
          ], 
          selectedSorting: '-1',
          parties: [
            { "id": "S",  "name": "Socialdemokratiska arbetarparti" },
            { "id": "M",  "name": "Moderata samlingspartiet" },
            { "id": "SD", "name": "Sverigedemokraterna	" },
            { "id": "C",  "name": "Centerpartiet" },
            { "id": "V",  "name": "Vänsterpartiet" },
            { "id": "KD", "name": "Kristdemokraterna" },
            { "id": "L",  "name": "Liberalerna" },
            { "id": "MP", "name": "Miljöpartiet de Gröna" }
          ],
          selectedParty: '-1',
          reps: [], /* reps is short for representatives */
          votes: []
      }
  },
  methods: {  
    sortListing(event, selectedSorting) { 
        this.selectedSorting = selectedSorting;
        let reps = this.reps; // sorting is faster with local object array
        this.reps = [];
        switch(selectedSorting) {
            case "fodd_ar":         reps.sort(function(a,b) {return (a.fodd_ar > b.fodd_ar) ? 1 : ((b.fodd_ar > a.fodd_ar) ? -1 : 0);} ); break; // birth year
            case "kon":             reps.sort(function(a,b) {return (a.kon > b.kon) ? 1 : ((b.kon > a.kon) ? -1 : 0);} ); break; // gender
            case "sorteringsnamn":  reps.sort(function(a,b) {return (a.sorteringsnamn > b.sorteringsnamn) ? 1 : ((b.sorteringsnamn > a.sorteringsnamn) ? -1 : 0);} ); break; // last name
            case "valkrets":        reps.sort(function(a,b) {return (a.valkrets > b.valkrets) ? 1 : ((b.valkrets > a.valkrets) ? -1 : 0);} ); break; // voting district
        }  
        this.reps = reps;      
    },
    getRepresentatives() {
        axios.get('https://data.riksdagen.se/personlista/?utformat=json')
        .then((response) => {                     
            /* keep only needed  data in array */   
            let reps = response.data.personlista.person;
            console.log('Representatives: ', reps);   
            let tempReps = [];            
            reps.forEach(function (item) {
                tempReps.push({
                    "intressent_id" : item.intressent_id,
                    "bild_url_80" : item.bild_url_80,
                    "parti" : item.parti,
                    "valkrets" : item.valkrets,
                    "fodd_ar" : item.fodd_ar,
                    "kon" : item.kon,
                    "sorteringsnamn" : item.sorteringsnamn,
                    "status" : item.status,
                    "valkrets" : item.valkrets
                });
            });
            this.reps = tempReps;
        }).catch((error) => {            
            console.log(error);
        });
    },
    getVotes: async function () {
        await axios.get('https://data.riksdagen.se/voteringlista/?rm=2019%2F20&sz=500&utformat=json')
        .then((response) => {            
            this.votes = response.data.voteringlista.votering;              
            
            /* keep only needed  data in array */   
            let votes = response.data.voteringlista.votering;              
            let tempVotes = [];            
            let i;
            let votesLen = this.votes.length;
            for (i = 0; i < votesLen; i++) {
                tempVotes.push({ "intressent_id" : votes[i].intressent_id, "rost" : votes[i].rost });
            }
            this.votes = tempVotes;
            /* Possible TODO: push these values in reps array */
        }).catch((error) => {
            console.log(error);
        });
    },
    // returns number of votes for: Ja Nej Avstår
    getRepVotes(intressent_id, voteType) {
        let votes = this.votes; // temporary array used for faster searching
        let numVotes = 0;
        if(votes.find(v => v.intressent_id == intressent_id)) {
            switch (voteType) {
                case 'Ja': numVotes = votes.reduce(function (n, vote) {
                        return n + (vote.intressent_id == intressent_id && vote.rost == 'Ja');
                    }, 0); break;
                case 'Nej': numVotes = votes.reduce(function (n, vote) {
                        return n + (vote.intressent_id == intressent_id && vote.rost == 'Nej');
                    }, 0); break;
                case 'Avs': numVotes = votes.reduce(function (n, vote) {
                        return n + (vote.intressent_id == intressent_id && vote.rost == 'Avstår');
                    }, 0); break;
            }
        }
        return numVotes; 
    }
  }, 
  filters: {
    displayFullname(str) {
        let arr = str.split(",");
        return arr[1].trim() + ' ' + arr[0].trim();
    }
  },
  created() {
    this.getRepresentatives();
    this.getVotes();
  }
}
</script>

<style>
#listing {
    margin-top: 40px;
}
#listing .column {
    background-color: #d6cfcf;
    border-radius: 14px;
    border: solid 1px #bdb9b9;
}
#listing .table {
    background-color: #f7f7f7;
    color: #613f3f;
}
small {
    font-size: .65em;
    text-transform: uppercase;
    color: #a1a1a1;
}
.table th {
    padding: 26px 0 0 0;
    background-color: #e3dfdf;    
    text-transform: uppercase;
    color: #908c8c;
    letter-spacing: .06em;
}
.table td, .table th {
    padding: .5em .75em .3em .75em;
}
.table .voteNumsLabel {
    float: right;
    margin-right: 15px;
    word-spacing: 14px;
}
.table td img {
    margin: 3px 0;
    border: solid 1px #cecece;
    border-bottom: none;
    border-radius: 2px;
}
.table .birthYearTableHeader {
  float: right;
}
.table .age {
    padding: 0 8px;
    background-color: #d1cccc;
    border-radius: 13px;
    letter-spacing: .03em;
    border: solid 1px #b3afaf;
    margin-left: 8px;
    float: right;
}
.table .party {
    text-align: center;
    font-weight: 600;
    font-size: 17px;
    color: #908c8c;
}
.table td.voteNumWrapper {
    padding-top: 11px;
}
.table td .voteNum {
    padding: 4px 12px 10px 12px;
    font-size: 48px;
    border-radius: 6px;
    font-weight: 400;
    margin: 6px 1px;
}
.table td .ja {
    background-color: #ccf1cc;
    color: #639063;
}
.table td .nej {
    background-color: #ec9f9f;
    color: #a74747;
}
.table td .avs {
    background-color: #e0d98b;
    color: #958802;
}
.loader-wrapper {
    position: absolute;
    top: 200px;
    left: 0;
    height: 100%;
    width: 100%;
    background: #fff;
    z-index: -1;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 6px;
}
.loader-wrapper .loader {
    height: 80px;
    width: 80px;
}
</style>