<template>
	<div>
		<h2 class="mb-4">
			Articles 
			<button type="button" class="btn btn-primary"  data-toggle="modal" @click="clearFields()" data-target="#myModal">
			  <i class="fa fa-plus"></i> Add Article
			</button>
			<button style="display:none !important" type="button" ref="btn"  class="btn btn-primary" data-toggle="modal" data-target="#myModal">
			  <i class="fa fa-plus"></i> Add Article
			</button>
		</h2>
		  <div class="modal" id="myModal">
		  <div class="modal-dialog">
		    <div class="modal-content">

		      <!-- Modal Header -->
		      <div class="modal-header">
		        <h4 class="modal-title">Modal Heading</h4>
		        <button type="button" ref="close" class="close" data-dismiss="modal">&times;</button>
		      </div>
		      <div class="modal-body">
				<form class="mb-3" @submit.prevent="addArticle()">
					<div class="form-group">
						<input type="text" class="form-control" placeholder="Title" v-model="article.title">
					</div>
					<div class="form-group">
						<textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
					</div>
					<button class="btn btn-light btn-block" type="submit">Save</button>
				</form>
			  </div>

			      <!-- Modal footer -->
			      <div class="modal-footer">
			        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
			      </div>

			    </div>
			  </div>
			</div>
		<hr>
		<nav aria-label="Page navigation example">
		  <ul class="pagination">
		    <li :class="{disabled:!pagination.prev_page_url}" class="page-item unselectable"><a class="page-link" href="#" @click.prevent="fetchArticles(pagination.prev_page_url)">Previous</a></li>
		    <li class="page-item disabled unselectable"><a class="page-link text-dark" href="#">Page {{pagination.current_page}} of {{pagination.last_page}}</a></li>		  
		    <li :class="{disabled:!pagination.next_page_url}" class="page-item unselectable"><a class="page-link" href="#" @click.prevent="fetchArticles(pagination.next_page_url)">Next</a></li>
		  </ul>
		</nav>
		<div class="card card-body mb-2" v-for="article in articles">
			<h3>{{article.title}}</h3>
			<p>{{article.body}}</p>
			<hr>
			<div class="row">
				<div class="col-md-6">
					<button class="btn btn-primary" style="width:150px;" @click="editArticle(article)"><i class="fa fa-pencil"></i>&nbsp;Edit</button>
					<button class="btn btn-danger" style="width:150px;" @click="deleteArticle(article.id)"><i class="fa fa-trash"></i>&nbsp;Delete</button>	
				</div>
				
			</div>
		</div>
	</div>
</template>

<script type="text/javascript">
toastr.options = {
				  "debug": false,
				  "positionClass": "toast-bottom-right",
				  "onclick": null,
				  "fadeIn": 300,
				  "fadeOut": 1000,
				  "timeOut": 5000,
				  "extendedTimeOut": 1000
				};
	export default{
		data(){
			return {
				articles:[],
				article:{
					id:'',
					title:'',
					body:''
				},
				article_id:'',
				pagination:{},
				edit: false
			}
		}, 
		created(){
			this.fetchArticles();
		},
		methods: {
			fetchArticles(page_url){
				let vm = this;
				page_url = page_url || '/api/articles';
				fetch(page_url)
					.then(res => res.json())
					.then(res => {
						this.articles = res.data;
						vm.makePagination(res.meta,res.links);
					})
					.catch(err => console.log(err));
			},
			makePagination(meta,links){
			 	let pagination = {
				 	current_page: meta.current_page,
					last_page: meta.last_page,
					next_page_url: links.next,
					prev_page_url: links.prev,	
			 	}
			 	this.pagination = pagination;
				
			},
			addArticle(){
				
				if(this.article.title == '' || this.article.body == ''){
					return;
				}
				if (this.edit === false){
					//Add
					fetch('/api/article',{
						method:'post',
						body: JSON.stringify(this.article),
						headers: {
							'content-type':'application/json'
						}
					})
					.then(res => res.json())
					.then(data => {
						this.article.title = "";
						this.article.body = "";
						this.article.id = "";
						this.edit = false;
						toastr.success('Article added');
						this.fetchArticles();
					})
					.catch(err => console.log(err))
				}else{
					//Edit
					fetch('/api/article',{
						method:'put',
						body: JSON.stringify(this.article),
						headers: {
							'content-type':'application/json'
						}
					})
					.then(res => res.json())
					.then(data => {
						// this.article.id = "";
						delete this.article.id;
						delete this.article.article_id;
						this.article.title = "";
						this.article.body = "";
						this.edit = false;						
						toastr.info('Article updated');
						this.fetchArticles();
					})
					.catch(err => console.log(err))
				}
				this.$refs.close.click();
			},
			editArticle(article){
				this.edit = true;
				this.article.id = article.id;
				this.article.article_id = article.id;
				this.article.title = article.title;
				this.article.body = article.body;
				this.$refs.btn.click();
			},
			deleteArticle(id){
				if(confirm('Are you sure?')){
					fetch(`/api/article/${id}`,{
						method: 'delete'
					})
					.then(res => res.json())
					.then(data => {
						toastr.error('Article removed');
						this.fetchArticles();
					})
					.catch(err => console.log(err));
				}
			},
			clearFields(){
				
					delete this.article.id;
					delete this.article.article_id;
					this.article.title = "";
					this.article.body = "";
					this.edit = false;	
				
			}
		}
	}
</script>