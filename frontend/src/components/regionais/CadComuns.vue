<template>
<div class="comuns">
    <div class="comum-admin" >
        <b-form v-if="isEdit">
            <input id="comum-id" type="hidden" v-model="comum.id" />
            <b-row>
                <b-col md="4" sm="12">
                    <b-form-group label="Regional:" label-for="comum-name">
                        <b-form-select id="regional-comum" :readonly="mode === 'remove'"
                        v-model="comum.idregional" :options="regionais"> 
                        </b-form-select>
                    </b-form-group>
                </b-col>
                <b-col md="4" sm="12">
                    <b-form-group label="Administração:" label-for="comum-name">
                        <b-form-select id="administracao-comum" :readonly="mode === 'remove'"
                        v-model="comum.idadministracao" :options="FilteredAdministracoes"> 
                        </b-form-select>
                    </b-form-group>
                </b-col>
                <b-col md="4" sm="12">
                    <b-form-group label="Nome da comum:" label-for="comum-name">
                        <b-form-input id="comum-name" type="text"
                            v-model="comum.nome" required
                            :readonly="mode === 'remove'"
                            placeholder="Informe o Nome da comum..." />
                    </b-form-group>
                </b-col>
            </b-row>
            <br>
            <b-row>
                <b-col xs="12">
                    <b-button variant="primary" v-if="mode === 'save'"
                        @click="save">Salvar</b-button>
                    <b-button variant="danger" v-if="mode === 'remove'"
                        @click="remove">Excluir</b-button>
                    <b-button class="ml-2" @click="reset">Cancelar</b-button>
                </b-col>
            </b-row>
        </b-form>
        <hr>
        <div v-show="!isEdit">
            <div class="card-header">
                <h3>Comuns
                    <button type="button" @click="isEdit = true" class="btn btn-outline-info btn-lg" v-if="isAdmin">
                        <i class="fa fa-plus-circle"></i>
                    </button>
                    <span>&nbsp;</span>
                    <BotaoDownloadExcel 
                        :dados="dadosPlanilha"
                        planilha="Comuns"
                        arquivo="Controle de Setores - Comuns.xlsx"
                    />
                </h3>
            </div>
            <div >
            <vue-good-table
                :pagination-options="{
                    enabled: true,
                    mode: 'records',
                    perPage: 5,
                    position: 'bottom',
                    perPageDropdown: [3, 5, 10],
                    dropdownAllowAll: true,
                    setCurrentPage: 1,
                    jumpFirstOrLast : true,
                    firstLabel : 'Primeira',
                    lastLabel : 'Última',
                    nextLabel: 'Próxima',
                    prevLabel: 'Anterior',
                    rowsPerPageLabel: 'Linhas por página',
                    ofLabel: 'de',
                    pageLabel: 'Página', // for 'pages' mode
                    allLabel: 'Todas',
                }"
                ref="comuns"
                @on-selected-rows-change="selectionChanged"
                :columns="columns"
                :rows="comuns"
                :select-options="{ enabled: true, 
                    selectionText: 'Linhas selecionadas',
                    disableSelectInfo: true}"
                :search-options="{ enabled: true, placeholder: 'Procurar...', }"
                styleClass="vgt-table striped hover">
                <template slot="table-row" slot-scope="data">
                    <span v-if="data.column.field == 'actions'">
                        <div v-if="isAdmin"> 
                            <b-button variant="warning" @click="loadcomum(data.row)" class="mr-2">
                                <i class="fa fa-pencil"></i>
                            </b-button>
                            <b-button variant="danger" @click="loadcomum(data.row, 'remove')">
                                <i class="fa fa-trash"></i>
                            </b-button>
                        </div>
                        <div v-else>
                            --
                        </div>
                    </span>
                    <span v-if="data.column.field == 'idadministracao'">
                        <span>{{findAdministracao(data.formattedRow[data.column.field])}}</span>
                    </span>
                    <span v-if="data.column.field == 'idregional'">
                        <span>{{findRegional(data.formattedRow[data.column.field])}}</span>
                    </span>
                    <span v-if="data.column.field != 'idregional' && data.column.field != 'idadministracao' && data.column.field != 'actions'">
                    {{data.formattedRow[data.column.field]}}
                    </span>
                </template>

            </vue-good-table>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import PageTitle from '../tamplate/PageTitle.vue'
import { baseApiUrl, showError } from '@/global'
import axios from 'axios'
import moment from 'moment';
import 'vue-good-table/dist/vue-good-table.css'
import { VueGoodTable } from 'vue-good-table';
import { mapState } from 'vuex'
import BotaoDownloadExcel from '../exportacao/BotaoDownloadExcel.vue'

    
export default {
    name: 'comuns',
    components:{PageTitle, VueGoodTable, BotaoDownloadExcel},
    computed: {
        dadosPlanilha() {
            const dados = []
            this.comuns.forEach((obj) => {
                let comum = {};
                comum['RA - Regional Administrativa'] = this.findRegional(obj.idregional);
                comum['ADM - Administração'] = this.findAdministracao(obj.idadministracao);
                comum['Setor - Comum'] = obj.nome;
                dados.push(comum);
            })
            return dados
        },
        ...mapState(['isAdmin'])
    } ,
    data: function() {
        return {
            mode: 'save',
            comum: {},
            comuns: [],
            administracoes: [],
            regionais: [],
            FilteredAdministracoes: [],
            isEdit: false,
            selectionChanged: [],
            columns: [
                {label: 'RA - Regional Administrativa',field: 'idregional',},
                {label: 'ADM - Administração',field: 'idadministracao',},
                {label: 'Setor - Comum',field: 'nome',},
                {label: 'Ações',field: 'actions',} 
      ],
        }
    },
    methods: {

        loadcomuns() {
            const url = `${baseApiUrl}/comumComAdministracao`
            axios.get(url).then(res => {
                this.comuns = res.data
            })
        },
        async loadRegionais() {
            const url = `${baseApiUrl}/regional`
            await axios.get(url).then(res => {
                this.regionais[0] = {}
                this.regionais[0].value = null
                this.regionais[0].text = 'Selecione a Regional'
                for (let index = 1; index <= res.data.length; index++) {
                    this.regionais[index] = {}
                    this.regionais[index].value = res.data[index-1].id
                    this.regionais[index].text = res.data[index-1].nome
                }
            })
        },
        async loadAdministracoes() {
            const url = `${baseApiUrl}/administracao`
            await axios.get(url).then(res => {
                this.administracoes[0] = {}
                this.administracoes[0].value = null
                this.administracoes[0].text = 'Selecione a Regional'
                for (let index = 1; index <= res.data.length; index++) {
                    this.administracoes[index] = {}
                    this.administracoes[index].value = res.data[index-1].id
                    this.administracoes[index].text = res.data[index-1].nome
                    this.administracoes[index].idregional = res.data[index-1].idregional
                }
            })
        },
        reset() {
            this.isEdit = false
            this.mode = 'save'
            this.comum = {}
            this.loadcomuns()
        },
        save() {
            const method = this.comum.id ? 'put' : 'post'
            const id = this.comum.id ? `/${this.comum.id}` : ''
            delete this.comum.vgt_id;
            delete this.comum.vgtSelected
            delete this.comum.originalIndex;
            delete this.comum.idregional;
            axios[method](`${baseApiUrl}/comum${id}`, this.comum)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        remove() {
            delete this.comum.vgt_id
            delete this.comum.originalIndex
            const id = this.comum.id
            axios.delete(`${baseApiUrl}/comum/${id}`)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        findAdministracao(value){
            if(this.administracoes.length>0){
                let item = this.administracoes.find(item=>item.value==value)
                return item.text
            }  
        },
        findRegional(value){
            if(this.regionais.length>0){
                let item = this.regionais.find(item=>item.value==value)
                return item.text
            }  
        },
        loadcomum(comum, mode = 'save') {
            console.log(comum)
            this.mode = mode
            this.comum = { ...comum }
            this.isEdit = true
        }
        
    },
     watch:{
        'comum.idregional': function(newVal, oldVal){
            this.FilteredAdministracoes = this.administracoes.filter((item) => item.idregional == newVal)
        },
        
    },
    async mounted() {

        await this.loadAdministracoes()
        await this.loadRegionais()
        this.loadcomuns()
    }
}
</script>

<style>
</style>
