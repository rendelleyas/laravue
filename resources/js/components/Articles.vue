<template>
    <div>
        <h2>Articles</h2>
         <form @submit.prevent="addArticle" class="mb3">
            <div class="form-group">
                <input type="text" class="form-control" name="title" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <input type="text" class="form-control" name="body" placeholder="Body" v-model="article.body">
            </div>
            <button type="submit" class="btn btn-info btn-block mb-2">Save</button>
        </form>
        <button @click="clearForm" class="btn btn-danger btn-block">Clear</button>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <!-- ma disabled sya og way sud ang previos -->
                <li class="page-item" v-bind:class="[{ disabled: !pagination.prev_page_url}]">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Prev</a>
                </li>
                <li class="page-item"><a class="page-link text-dark">Page {{ pagination.current_page}} of {{ pagination.last_page}}</a></li>
                <li class="page-item" v-bind:class="[{ disabled: !pagination.next_page_url}]">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
            </ul>
        </nav>
         <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
             <h3> {{ article.title}}</h3>
             <p> {{ article.body }}</p>
             <hr>
             <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button>
             <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
         </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            }
        },

        //everytime ma load ang site always niya ni tawagun
        created(){
            this.fetchArticles();
        },


        methods: {
            fetchArticles(page_url){ //page_url - kung aha ka na page
                let vm = this;
                page_url = page_url || '/api/articles';
                fetch(page_url) //diri mag kuha og data
                .then(res => res.json()) //e json niya
                .then(res =>{
                    this.articles = res.data; //gi set ang declared var=article as the data from db
                    vm.makePagination(res.meta, res.links); // gi set niya ang mga value sa pagination n variable
                })
                .catch(err => console.log(err))
            },

            makePagination(meta, links){
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                }

                this.pagination = pagination;
            },

            //unsaon pag delete mao ni.
            deleteArticle(id){
                if(confirm('Are you sure?')) {
                    fetch('api/article/' + id,{
                        method: 'delete'
                    })
                    .then(res => res.json())
                    .then(data =>{
                        alert('Article Removed');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
                }
            },

            addArticle(){
                if(this.edit === false){
                    console.log(this.article);
                    //Add
                    fetch('/api/article', {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers: {
                            'Accept': 'application/json',
                            'Content-Type': 'application/json',
                        }
                    })
                    .then(res => res.json())
                    .then(data =>{
                        alert('Article Added');
                        this.article.title = '';
                        this.article.body = '';
                        this.fetchArticles();
                    })

                }else{
                    // Update
                    fetch('/api/article', {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers: {
                            'Accept': 'application/json',
                            'Content-Type': 'application/json',
                        }
                    })
                    .then(res => res.json())
                    .then(data =>{
                        alert('Article Updated');
                        this.clearForm();
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
                }

            },

            editArticle(article){
                this.edit = true;
                this.article.id =article.id;
                this.article.article_id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            },

             clearForm() {
                this.edit = false;
                this.article.id = null;
                this.article.article_id = null;
                this.article.title = '';
                this.article.body = '';
            }
        }
    }
</script>

<style lang="scss" scoped>

</style>
