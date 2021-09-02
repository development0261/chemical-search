<template>
	<div id="app">
		<div class="container">
			<div class="row">
				<div class="col-md-6">
					<div class="form-group">
						<label>Input option : </label>
						<textarea class="form-control" v-model="searchText" rows="3" cols="40" @keyup="updateTextFormat"></textarea>
						<div v-if="searchText" class="mt-2">Text : {{ searchText }}</div>
						<button data-toggle="modal" data-target="#myModal" @click="setModalserachText" class="mt-2 btn btn-outline-dark">Search</button>
					</div>
				</div>
				<div class="col-md-6">
					<div v-if="ResultKeywords" class="form-group">
						<label>On Submit : </label>
						<textarea v-model="ResultKeywords" rows="3" cols="40" class="form-control"></textarea>
					</div>
				</div>
			</div>
		</div>
		<div class="container">
			<div class="modal" id="myModal" role="dialog">
				<div class="modal-dialog  modal-dialog-centered">
					<!-- Modal content-->
					<div class="modal-content">
						<div class="modal-header p-1 justify-content-end">
							<button type="button" class="close btn" data-dismiss="modal">&times;</button>
							<!-- <h4 class="modal-title">Modal Header</h4> -->
						</div>
						<div class="modal-body">
							<div> 
								<label>Chemical Synanyms : </label>
								<textarea class="form-control" disabled v-model="ModalsearchText" rows="2" cols="40"></textarea>
								<button class="btn btn-success mt-2" @click="AnalyzeKeywords">Analyze</button>
							</div>
							<div class="mt-3"><b v-if="apiresponse.length">Keywords Suggestions : </b></div>
							<div class="text-center mt-2" v-if="loading">
								<p>Loading...</p>
							</div>
							<div class="words-block" v-else>
								<div v-for="(row, index) in apiresponse" :key="index">
									<input :id="row" type="checkbox" name="Keywords" :value="row" v-model="checkedKeywords" ref="suggestionKeyword"/>
									<label @click="addWord(row)" :for="row" :class="{'active': checkedKeywords.some(x => x.toLowerCase() == row.toLowerCase())}"  >{{row}}</label>
								</div>
							</div>
						</div>
						<div class="modal-footer">
							<button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
							<button type="button" class="btn btn-primary" @click="clearSelection" data-dismiss="modal">Clear</button>
							<button type="button" class="btn btn-success" data-dismiss="modal" @click="SubmitCheckedKeywords">Submit</button>
						</div>
					</div>
					
				</div>
			</div>
		</div> 
	</div>
</template>

<script>

export default {
	name: 'app',
	data () {
		return {
			query: "",
			searchText: '',
			ModalsearchText: '',
			apiresponse: [],
			checkedKeywords:[],
			word_array: [],
			loading: false,
			ResultKeywords:''
		}
	},
	methods: {
		clearSelection: function() {
			this.searchText = "";
			this.ResultKeywords = '';
			this.ModalsearchText = '';
			this.apiresponse = [];
			this.word_array = [];
			this.checkedKeywords = [];
		},
		setModalserachText: function () {
			let word = this.searchText.split(/[OR,AND]/);
			const arrFiltered = word.filter(el => {
				return el != null && el != '';
			});
			this.word_array = arrFiltered.map(function (el) {
				let new_l = el.trim();
				new_l = new_l.replace("'", '');
				new_l = new_l.replaceAll("\"", '');
				return new_l;
			});
			this.ModalsearchText = this.word_array.join(" ");
		},
		async AnalyzeKeywords() {
			this.loading = true;
			this.checkedKeywords = [];
			this.apiresponse = [];
			this.ResultKeywords = '';
			console.log(this.word_array)
			let promiseArr = this.word_array.map(word => fetch('https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/'+word+'/synonyms/json').then(res => res.json()));
			Promise.all(promiseArr).then(res => {		
				res.forEach(r => {
					if ('InformationList' in r) {							
						this.apiresponse = this.apiresponse.concat(r.InformationList.Information[0].Synonym);
						this.word_array.forEach(word => {
							this.checkedKeywords.push(word);
						});
					}
				})
				this.loading = false;
			})
		},
		addWord(word) {
			if(this.word_array.some(x => x.toLowerCase() == word.toLowerCase())) {
				var word_index = this.word_array.findIndex(item => word.toLowerCase() === item.toLowerCase());				
				if ( ~word_index ) this.word_array.splice(word_index, 1);
			} else {
				this.word_array.push(word);				
			}
		},
		SubmitCheckedKeywords: function(){
			this.checkedKeywords = this.checkedKeywords.filter( function( item, index, checkedKeywords ) {
           		return checkedKeywords.indexOf(item) == index;
    		});
			var resultkeywordss = this.checkedKeywords.map(function(keyword) {
				return '(' + keyword + ')';
			});

			// join the fruits with the comma's in between
			var result = resultkeywordss.join(' OR ');			
			this.ResultKeywords = result;
		},
		updateTextFormat: function() {
			this.searchText = this.searchText.replace(regex, '');
			const regex = /\bor\b/ig;
			this.searchText = this.searchText.replace(regex, 'OR');
			const regex1 = /\band\b/ig;
			this.searchText = this.searchText.replace(regex1, 'AND');	
			var matches = this.searchText.match(/"/g);
			if(matches) {
				if(matches.length >= 2) {
					var doubleQuotedString = extractAllText(this.searchText);
					doubleQuotedString.forEach(e => {
						var orString = e.replace(regex, 'or');
						var demo = orString.replace(regex1, 'and');
						this.searchText = this.searchText.replaceAll(e,demo);						
					});
				}
			}
			function extractAllText(str) {
				const re = /"(.*?)"/g;
				const result = [];
				let current;
				while (current = re.exec(str)) {					
					result.push(current.pop());
				}
				return result.length > 0
					? result
					: [str];
			}
		}
	},
}
</script>
<style>
textarea {
	resize: none;
}
a {
	color: #42b983;
}
.fade.in {
	opacity: 1 !important;
}
.modal-backdrop.fade {
	opacity: 0.5 !important;
}
.modal.in .modal-dialog
{
	transform: translate(0,0) !important;
}
.words-block::-webkit-scrollbar{
	width: 5px;
    background-color: #F5F5F5;
}
.words-block::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgb(0 0 0 / 30%);
    background-color: #ced4da;
}
.words-block {
	display: flex;
    overflow-y: scroll;
    max-height: 175px;
    flex-wrap: wrap;
}
.words-block div {
	position: relative;
    margin-right: 10px;
    margin-top: 10px;
}
.words-block div input {
    visibility: hidden;
    position: absolute;
}
.words-block div label {
	cursor: pointer;
	user-select: none;
    padding: 5px 10px;
    background: #e3e3e3;
    border-radius: 3px;
}
.words-block div label.active {
	background: #00bcd4;
	color: #fff;
}
</style>
