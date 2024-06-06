<template>
    <AuthenticatedLayout>
        <section style="background-color: #eee;">
            <div class="container py-5">
                <div class="row">
                    <div class="col-md-6 col-lg-5 col-xl-4 mb-4 mb-md-0">

                        <h5 class="font-weight-bold mb-3 text-center text-lg-start">Member</h5>

                        <div class="card">
                            <div class="card-body">

                                <ul class="list-unstyled mb-0">
                                    <li v-for="user in users" class="p-2 border-bottom" style="background-color: #eee;">
                                        <div class="d-flex flex-row">
                                            <div class="pt-1">
                                                <p class="fw-bold mb-0">{{ user.name }}</p>
                                            </div>
                                        </div>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6 col-lg-7 col-xl-8">
                        <h3 class="text-gray-700 mb-4 text-lg">{{ chat.title ?? 'Your chat' }}</h3>
                        <ul v-if="messages" class="list-unstyled">
                            <li v-if="!isLastPage" class="text-center mb-2">
                                <a @click.prevent="getMessages"
                                   class="inline-block bg-sky-600 text-white text-xs px-3 py-2 rounded-lg" href="#">Load more</a>
                            </li>
                            <li v-for="message in messages.slice().reverse()" class="d-flex justify-content-between mb-4">
                                <img v-if="!message.is_owner" src="https://mdbcdn.b-cdn.net/img/Photos/Avatars/avatar-8.webp" alt="avatar"
                                     class="rounded-circle d-flex align-self-start ms-3 shadow-1-strong" width="60">
                                <div class="card w-100">
                                    <div class="card-header d-flex justify-content-between p-3">
                                        <p class="fw-bold mb-0">{{ message.user_name }}</p>
                                        <p class="text-muted small mb-0"><i class="far fa-clock"></i>{{message.time}}</p>
                                    </div>
                                    <div class="card-body">
                                        <p class="mb-0">
                                            {{ message.body }}
                                        </p>
                                    </div>
                                </div>
                                <img v-if="message.is_owner" src="https://mdbcdn.b-cdn.net/img/Photos/Avatars/avatar-8.webp" alt="avatar"
                                     class="rounded-circle d-flex align-self-start ms-3 shadow-1-strong" width="60">
                            </li>
                            <li class="bg-white mb-3">
                                <div data-mdb-input-init class="form-outline">
                                    <textarea v-model="body" class="form-control" placeholder="Message" ></textarea>
                                </div>
                            </li>
                            <li>
                                <a @click.prevent="store"
                                   class="btn btn-info btn-rounded float-end" href="#">Send
                                </a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </section>

    </AuthenticatedLayout>
</template>

<script>

import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";

export default {
    name: "Show",
    components: {AuthenticatedLayout},

    props: [
        'chat',
        'users',
        'messages',
        'isLastPage'
    ],

    created() {
        window.Echo.channel(`store-message.${this.chat.id}`)
            .listen('.store-message', res => {
                this.messages.unshift(res.message)
                if (this.$page.url === `/chats/${this.chat.id}`
                ) {
                    axios.patch('/message_statuses', {
                        message_id: res.message.id,
                        user_id: this.$page.props.auth.user.id
                    })
                        .catch(error => {

                        });
                }
            })
    },

    data() {
        return {
            body: '',
            page: 1,
        }
    },


    computed: {
        userIds() {
            return this.users.map(user => {
                return user.id
            }).filter(userId => {
                return userId !== this.$page.props.auth.user.id
            })
        },
    },


    methods: {
        store() {
            axios.post('/messages', {
                chat_id: this.chat.id,
                body: this.body,
                user_ids: this.userIds
            })
                .then(res => {
                    this.messages.unshift(res.data)
                    this.body = ''
                })
        },

        getMessages() {
            axios.get(`/chats/${this.chat.id}?page=${++this.page}`)
                .then(res => {
                    this.messages.push(...res.data.messages)
                    this.$page.props.isLastPage = res.data.is_last_page
                })
        },
    }

}

</script>

<style scoped>

</style>
