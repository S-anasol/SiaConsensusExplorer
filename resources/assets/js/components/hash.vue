<style scoped>
.btn-collapse {
    border-bottom: 0px;
    border-radius: 4px 4px 0px 0px;
}

.alert {
    border-radius: 0px 4px 4px 4px;
}

.flying {
    position: fixed;
    bottom: 0px;
    right: 20px;
}
</style>
<template>
<div>
    <div class="row" v-if="hash.hash">
        <div class="col-md-8 col-md-offset-2">
            <h3>Hash: <router-link :to="'/hash/'+hash.hash">{{hash.hash}}</router-link></h3>
            <h5>Hash type: {{hash.type | hashType}}</h5>
            <h5 v-show="spendable">Spent: {{(hash.spent == 1) ? "Yes":"No"}}</h5>
            <h5>Total SC Outputs: {{scoutputs | currency}} SC</h5>
            <h5>Total SF Outputs: {{sfoutputs | sfCurrency}} SF</h5>
            <h5>Found in {{uniqueBlocks.length}} block(s) <span class="label label-danger" v-if="uniqueBlocks.length >= 10000">Reached 10000 blocks limit, older blocks will not be loaded.</span></h5>
        </div>
    </div>
    <div class="flying alert alert-success">
        <h5>Hash type: {{hash.type | hashType}}</h5>
        <h5 v-show="spendable">Spent: {{(hash.spent == 1) ? "Yes":"No"}}</h5>
        <h5>Total SC Outputs: {{scoutputs | currency}} SC</h5>
        <h5>Total SF Outputs: {{sfoutputs | sfCurrency}} SF</h5>
        <h5>Found in {{uniqueBlocks.length}} block(s)</h5>
    </div>

    <div class="row" v-if="loading">
        <div class="col-md-8 col-md-offset-2">
            <div class="progress">
                <div class="progress-bar progress-bar-striped active" style="width: 100%">
                    <span>Loading...</span>
                </div>
            </div>
        </div>
    </div>

    <div class="row blocksList" v-for="block in orderedBlocks">
        <div class="col-md-8 col-md-offset-2">
            <button class="btn btn-primary btn-xs btn-collapse" type="button" data-toggle="collapse" :data-target="'#block'+block.height">
                    Block height #{{block.height}} // <span :title="moment.unix(block.headers.timestamp).format('MMMM DD YYYY, HH:mm:ss')">{{moment.unix(block.headers.timestamp).fromNow()}}</span>
            </button>
            <div class="collapse in" :id="'block'+block.height">
                <div class="row" v-if="block.minerpayouts">
                    <div class="col-md-12" v-for="scoutput in block.minerpayouts">
                        <div class="alert alert-info">
                            <p><span class="label label-success">Miner payout</span></p>
                            <p>Output ID:
                                <router-link :to="'/hash/'+scoutput.id">{{scoutput.id}}</router-link>
                            </p>
                            <p>Amount: {{scoutput.value | currency}} SC</p>
                            <p>Receiver:
                                <router-link :to="'/hash/'+scoutput.unlockhash">{{scoutput.unlockhash}}</router-link>
                            </p>
                        </div>
                    </div>
                </div>
                <div v-for="tr in block.transactions">
                    <div class="row" v-if="tr.siacoininputs">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="scoutput in tr.siacoininputs">
                                <p><span class="label label-success">Siacoin Input</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+scoutput.transaction">{{scoutput.transaction}}</router-link>
                                </p>
                                <p>Output ID:
                                    <router-link :to="'/hash/'+scoutput.id">{{scoutput.id}}</router-link>
                                </p>
                                <p>Receiver: {{scoutput.unlockconditions.publickeys[0].algorithm}}:{{scoutput.unlockconditions.publickeys[0].key}}</p>
                            </div>
                        </div>
                    </div>

                    <div class="row" v-if="tr.siacoinoutputs">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="scoutput in tr.siacoinoutputs">
                                <p><span class="label label-success">Siacoin Output</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+scoutput.transaction">{{scoutput.transaction}}</router-link>
                                </p>
                                <p>Output ID:
                                    <router-link :to="'/hash/'+scoutput.id">{{scoutput.id}}</router-link>
                                </p>
                                <p>Amount: {{scoutput.value | currency}} SC</p>
                                <p>Receiver:
                                    <router-link :to="'/hash/'+scoutput.unlockhash">{{scoutput.unlockhash}}</router-link>
                                </p>
                            </div>
                        </div>
                    </div>

                    <div class="row" v-if="tr.siafundinputs">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="scoutput in tr.siafundinputs">
                                <p><span class="label label-success">Siafund Input</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+scoutput.transaction">{{scoutput.transaction}}</router-link>
                                </p>
                                <p>Output ID:
                                    <router-link :to="'/hash/'+scoutput.id">{{scoutput.id}}</router-link>
                                </p>
                                <p>Receiver: {{scoutput.unlockconditions.publickeys[0].algorithm}}:{{scoutput.unlockconditions.publickeys[0].key}}</p>
                            </div>
                        </div>
                    </div>

                    <div class="row" v-if="tr.siafundoutputs">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="sfoutput in tr.siafundoutputs">
                                <p><span class="label label-success">Siafund Output</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+sfoutput.transaction">{{sfoutput.transaction}}</router-link>
                                </p>
                                <p>Output ID:
                                    <router-link :to="'/hash/'+sfoutput.id">{{sfoutput.id}}</router-link>
                                </p>
                                <p>Amount: {{sfoutput.value | sfCurrency}} SF</p>
                                <p>Receiver:
                                    <router-link :to="'/hash/'+sfoutput.unlockhash">{{sfoutput.unlockhash}}</router-link>
                                </p>
                            </div>
                        </div>
                    </div>

                    <div class="row" v-if="tr.filecontracts">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="filecontract in tr.filecontracts">
                                <p><span class="label label-success">File Contract</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+filecontract.transaction">{{filecontract.transaction}}</router-link>
                                </p>
                                <p>Contract ID:
                                    <router-link :to="'/hash/'+filecontract.id">{{filecontract.id}}</router-link>
                                </p>
                                <p>Payout: {{filecontract.payout | currency}} SC</p>
                                <p>Receiver:
                                    <router-link :to="'/hash/'+filecontract.unlockhash">{{filecontract.unlockhash}}</router-link>
                                </p>

                                <div class="row" v-if="filecontract.missedproofoutputs">
                                    <div class="col-md-12">
                                        <button class="btn btn-warning btn-xs btn-collapse" type="button" data-toggle="collapse" :data-target="'#missed'+filecontract.id">
                                                Missed Proofs
                                        </button>
                                        <div class="collapse" :id="'missed'+filecontract.id">
                                            <div class="alert alert-warning" v-for="(missedproof, id) in filecontract.missedproofoutputs">
                                                <p><span class="label label-warning">Missed Proof Output</span></p>
                                                <p>Output ID:
                                                    <router-link :to="'/hash/'+id">{{id}}</router-link>
                                                </p>
                                                <p>Amount: {{missedproof.value | currency}} SC</p>
                                                <p>Receiver:
                                                    <router-link :to="'/hash/'+missedproof.unlockhash">{{missedproof.unlockhash}}</router-link>
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                </div>

                                <div class="row" v-if="filecontract.validproofoutputs">
                                    <div class="col-md-12">
                                        <button class="btn btn-success btn-xs btn-collapse" type="button" data-toggle="collapse" :data-target="'#valid'+filecontract.id">
                                                Valid Proofs
                                        </button>
                                        <div class="collapse" :id="'valid'+filecontract.id">
                                            <div class="alert alert-success" v-for="(validproof, id) in filecontract.validproofoutputs">
                                                <p><span class="label label-success">Valid Proof Output</span></p>
                                                <p>Output ID:
                                                    <router-link :to="'/hash/'+id">{{id}}</router-link>
                                                </p>
                                                <p>Amount: {{validproof.value | currency}} SC</p>
                                                <p>Receiver:
                                                    <router-link :to="'/hash/'+validproof.unlockhash">{{validproof.unlockhash}}</router-link>
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="row" v-if="tr.filecontractrevisions">
                        <div class="col-md-12">
                            <div class="alert alert-info" v-for="filecontract in tr.filecontractrevisions">
                                <p><span class="label label-success">File Contract Revision #{{filecontract.newrevisionnumber}}</span></p>
                                <p>Transaction ID:
                                    <router-link :to="'/hash/'+filecontract.transaction">{{filecontract.transaction}}</router-link>
                                </p>
                                <p>Contract ID:
                                    <router-link :to="'/hash/'+filecontract.id">{{filecontract.id}}</router-link>
                                </p>
                                <p>New Filesize: {{filecontract.newfilesize | filesize}}
                                </p>

                                <div class="row" v-if="filecontract.newmissedproofoutputs">
                                    <div class="col-md-12">
                                        <button class="btn btn-warning btn-xs btn-collapse" type="button" data-toggle="collapse" :data-target="'#newmissed'+filecontract.id">
                                                Missed Proofs
                                        </button>
                                        <div class="collapse" :id="'newmissed'+filecontract.id">
                                            <div class="alert alert-warning" v-for="(missedproof, id) in filecontract.newmissedproofoutputs">
                                                <p><span class="label label-warning">Missed Proof Output</span></p>
                                                <p>Output ID:
                                                    <router-link :to="'/hash/'+id">{{id}}</router-link>
                                                </p>
                                                <p>Amount: {{missedproof.value | currency}} SC</p>
                                                <p>Receiver:
                                                    <router-link :to="'/hash/'+missedproof.unlockhash">{{missedproof.unlockhash}}</router-link>
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                </div>

                                <div class="row" v-if="filecontract.newvalidproofoutputs">
                                    <div class="col-md-12">
                                        <button class="btn btn-success btn-xs btn-collapse" type="button" data-toggle="collapse" :data-target="'#newvalid'+filecontract.id">
                                                Valid Proofs
                                        </button>
                                        <div class="collapse" :id="'newvalid'+filecontract.id">
                                            <div class="alert alert-success" v-for="(validproof, id) in filecontract.newvalidproofoutputs">
                                                <p><span class="label label-success">Valid Proof Output</span></p>
                                                <p>Output ID:
                                                    <router-link :to="'/hash/'+id">{{id}}</router-link>
                                                </p>
                                                <p>Amount: {{validproof.value | currency}} SC</p>
                                                <p>Receiver:
                                                    <router-link :to="'/hash/'+validproof.unlockhash">{{validproof.unlockhash}}</router-link>
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>


    <div class="row" v-if="chunkLoad">
        <div class="col-md-6 col-md-offset-3">
            <div class="progress">
                <div :class="'progress-bar '+((currentChunk < 10)?'progress-bar-striped active':'')" style="width: 100%">
                    <span>Loading {{currentChunk*10}} of {{chunks.length*10}} known blocks...</span>
                </div>
            </div>
        </div>
    </div>

    <div class="row" v-if="error">
        <div class="col-md-8 col-md-offset-2">
            <div class="alert alert-danger">
                <p v-for="err in error">{{err}}</p>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import hashType from '../filters/hashType.js'
import currency from '../filters/currency.js'
import sfCurrency from '../filters/sfCurrency.js'
import filesize from '../filters/filesize.js'
import _ from 'lodash'
import moment from 'moment'

export default {
    mounted() {
        this.load();
    },
    watch: {
        hash() {
            if (this.hash && typeof this.hash.blocks !== "undefined") {
                this.loadBlocks();
            }
        },
        "$route.params.hash": function() {
            this.load();
        }
    },

    created() {
        window.addEventListener('scroll', this.scrollCheck);
    },
    destroyed() {
        window.removeEventListener('scroll', this.scrollCheck);
    },

    computed: {
        orderedBlocks: function() {
            return _.orderBy(this.blocks, ['height'], ['desc']);
        },
        uniqueBlocks: function() {
            if (this.hash && typeof this.hash.blocks !== "undefined") {
                return _.uniqBy(this.hash.blocks, 'height');
            } else {
                return [];
            }
        },
        scoutputs: function(){
            var amount = 0;
            _.each(this.blocks, (b) => {
                _.each(b.minerpayouts, (o) => {
                    amount += parseInt(o.value);
                });
                _.each(b.transactions, (t) => {
                    _.each(t.siacoinoutputs, (o) => {
                        amount += parseInt(o.value);
                    });
                    _.each(t.filecontracts, (fc) => {
                        _.each(fc.validproofoutputs, (vp) => {
                            amount += parseInt(vp.value);
                        });
                    });
                });
            });

            return amount;
        },
        sfoutputs: function(){
            var amount = 0;
            _.each(this.blocks, (b) => {
                _.each(b.transactions, (t) => {
                    _.each(t.siafundoutputs, (o) => {
                        amount += parseInt(o.value);
                    });
                });
            });

            return amount;
        },
    },
    methods: {
        load() {
            if (this.loading) return false;

            this.hash = {};
            this.blocks = {};
            this.error = false;
            this.loading = true;
            axios.get('/api/hash/' + this.$route.params.hash)
                .then((response) => {
                    this.hash = response.data;
                    if(this.hash.type == 'siacoinoutputid' || this.hash.type == 'siafundoutputid') {
                        this.spendable = true;
                    }
                    this.loading = false;

                    if(this.hash.type == 'blockid') {
                        this.$router.push('/block/'+this.hash.blocks[0].height);
                    }
                })
                .catch((error) => {
                    console.log(error);
                    this.error = error.response.data;
                    this.loading = false;
                });
        },
        loadBlocks() {
            if (this.hash.blocks.length) {
                var blocks = _.uniq(_.flatMap(_.orderBy(this.hash.blocks, 'height', 'desc'), (b) => {
                    return b.height;
                }));
                this.chunks = _.chunk(blocks, 10);
                this.currentChunk = 0;
                this.loadChunk();
            }
        },
        loadChunk() {
            if (this.chunkLoad || this.currentChunk >= this.chunks.length) return false;

            this.chunkLoad = true;
            axios.post('/api/blocks/', {
                    blocks: this.chunks[this.currentChunk],
                    type: this.hash.type,
                    hash: this.hash.hash
                })
                .then((response) => {
                    this.currentChunk++;
                    _.each(response.data, (b, key) => {
                        this.$set(this.blocks, key, b);
                    })
                    this.chunkLoad = false;
                })
                .catch((error) => {
                    console.log(error);
                    this.chunkLoad = false;
                });
        },
        scrollCheck(e) {
            if (this.chunkLoad) return false;

            var el = this.$el.querySelector('.blocksList:last-child');

            var rect = el.getBoundingClientRect();
            var viewHeight = Math.max(document.documentElement.clientHeight, window.innerHeight);
            if (!(rect.bottom < 0 || rect.top - viewHeight >= 0)) {
                this.loadChunk();
            }
        },
        findTransaction() {
            switch (this.hash.type) {
                case 'filecontractid':
                    _.each(this.blocks, (b) => {
                        _.each(b.transactions, (o, trid) => {
                            var op_index = _.findKey(o.filecontracts, (fc, id) => {
                                return id == this.hash.hash;
                            });
                            if (op_index) {
                                var op = o.filecontracts[op_index];
                                op.id = op_index;
                                op.transaction = trid;
                                this.filecontract = op;
                            }
                        });
                    });
                    break;
            }
        }
    },
    filters: {
        hashType,
        currency,
        sfCurrency,
        filesize
    },
    data() {
        return {
            hash: {},
            blocks: {},
            chunks: [],
            currentChunk: 0,
            loading: false,
            chunkLoad: false,
            error: false,
            spendable: false,
            moment: moment,
        }
    }
}
</script>
