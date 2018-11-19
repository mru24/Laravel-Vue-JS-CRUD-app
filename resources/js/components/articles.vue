<template>
    <div>
        <h2 class="text-center my-4">Articles</h2>
        <hr>
        <form @submit.prevent="addArticle">
            <div class="form-group">
                <input class="form-control" type="text" v-model="article.title" placeholder="Post Title">
            </div>
            <div class="form-group">
                <textarea class="form-control" v-model="article.body" placeholder="Post Body"></textarea>
            </div>
            <button type="submit" class="btn btn-default text-success mx-auto">Save</button>
        </form>
        <hr>

        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li :class="[{disabled: !pagination.prev_page_url}]" class="page-item">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a>
                </li>
                <li class="page-item disabled">
                    <a class="page-link text-dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a>
                </li>
                <li :class="[{disabled: !pagination.next_page_url}]" class="page-item">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
            </ul>
        </nav>

        <div class="card card-body mb-2" v-for="(article, index) in articles" :key="index">
            <h3>{{ article.title }}</h3>
            <p>{{ article.body }}</p>
            <hr>
            <button class="btn btn-warning ml-auto" @click="editArticle(article)">Edit</button>
            <button class="btn btn-danger ml-auto" @click="startDeleteArticle(article.id)">Delete</button>
        </div>
        <transition name="modal">
            <div class="modalMessage" v-if="messageShow">
                <div class="modal-body">
                    <h4>{{ message }}</h4>
                </div>
                <div class="modal-footer">
                    <button v-if="!deletePost" class="btn btn-success ml-auto" @click="messageShow=false">Ok</button>
                    <button v-if="deletePost" class="btn btn-danger ml-auto" @click="deleteArticle(article.id)">Ok</button>
                </div>
            </div>
        </transition>

    </div>
</template>

<script>
export default {
    data () {
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            },
            article_id: '',
            pagination: {},
            edit: false,
            messageShow: false,
            message: '',
            deletePost: false
        }
    },
    created () {
        this.fetchArticles()
    },
    methods: {
        fetchArticles (page_url) {
            let vm = this
            page_url = page_url || 'api/articles'
            fetch(page_url)
                .then(res => res.json())
                .then(res => {
                    this.articles = res.data
                    vm.makePagination(res.meta, res.links)
                })
                .catch(err => console.log(err))
        },
        makePagination (meta, links) {
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }
            this.pagination = pagination
        },
        startDeleteArticle (postId) {
            this.article.id = postId
            this.messageShow = true
            this.message = "Are You Sure?"
            this.deletePost = true
        },
        deleteArticle (id) {
            this.messageShow = false         
            fetch(`api/article/${id}`, {
                method: 'delete'
            })
            .then(res => res.json())
            .then(data => {
                this.messageShow = true
                this.message = 'Article Removed'
                this.deletePost = false
                this.fetchArticles()
            })
            .catch(err => console.log(err))
        },
        addArticle () {
            if(this.edit === false) {
                // Add
                fetch('api/article', {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json)
                .then(data => {
                    this.article.title = ''
                    this.article.body = ''
                    this.messageShow = true
                    this.message = 'Article Added'
                    this.fetchArticles()
                })
                .catch(err => console.log(err))
            } else {
                // Update
                fetch('api/article', {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json)
                .then(data => {
                    this.article.title = ''
                    this.article.body = ''
                    this.messageShow = true
                    this.message = 'Article Edited'
                    this.fetchArticles()
                })
                .catch(err => console.log(err))
            }
        },
        editArticle (article) {
            this.edit = true
            this.article.id = article.id
            this.article.article_id = article.id
            this.article.title = article.title
            this.article.body = article.body
        },
        showModal () {
            this.messageShow = !this.messageShow
        }
    }
    
}
</script>

<style lang="sass">
button
    width: 120px
    margin-bottom: 5px
textarea
    resize: none
    height: 90px
.modalMessage
    position: fixed
    top: 10%
    left: 10%
    width: 80%
    background: white
    border: 1px solid #bbb
    border-radius: 5px
    transform: scale(1)
    box-shadow: 0 0 30px #999
    .modal-body
        text-align: center
        padding: 50px 0
    .modal-footer
        padding: 20px 10px


.modal-enter-active, .modal-leave-active
    transition: all .5s
.modal-enter, .modal-leave-to
    top: -50%
    transform: scale(.6)
    opacity: 0
    box-shadow: none
</style>


