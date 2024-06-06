<template>
    <AuthenticatedLayout>
        {{console.log(this.$page.props.auth)}}
        <section style="background-color: #eee;">
            <div v-if="users" class="container py-5">
                <div class="row">
                    <div class="col-md-6 col-lg-5 col-xl-4 mb-4 mb-md-0">

                        <h5 class="font-weight-bold mb-3 text-center text-lg-start">Users</h5>

                        <div class="card">
                            <div class="card-body">
                                <div class="flex items-center  mb-4 justify-between">
                                    <a v-if="!isGroup" @click.prevent="isGroup = true"
                                       class="inline-block bg-indigo-600 text-white text-xs px-3 py-2 rounded-lg"
                                       href="#">Make
                                        group</a>
                                    <div v-if="isGroup" class="flex items-center">
                                        <input class="h-8 mr-4 border border-gray-300 rounded-full" type="text"
                                               placeholder="group title"
                                               v-model="title">
                                        <a @click.prevent="storeGroup"
                                           :class="['inline-block mr-2  text-white text-xs px-3 py-2 rounded-lg',
                                               this.userIds.length > 1 ? 'bg-green-600' : 'bg-green-300']" href="#">Go
                                            chat</a>
                                        <a @click.prevent="refreshUserIds"
                                           class="inline-block bg-indigo-600 text-white text-xs px-3 py-2 rounded-lg"
                                           href="#">X</a>
                                    </div>
                                </div>

                                <ul class="list-unstyled mb-0">
                                    <li v-for="user in users" class="p-2 border-bottom" style="background-color: #eee;">
                                        <a href="#" class="d-flex justify-content-between">
                                            <div @click.prevent="store(user.id)" class="d-flex flex-row">
                                                <img src="https://mdbcdn.b-cdn.net/img/Photos/Avatars/avatar-8.webp"
                                                     alt="avatar"
                                                     class="rounded-circle d-flex align-self-center me-3 shadow-1-strong"
                                                     width="60">
                                                <div class="pt-1">
                                                    <p class="fw-bold mb-0">{{ user.name }}</p>
                                                </div>
                                            </div>
                                            <div v-if="isGroup" class="pt-1">
                                                <input @click="toggleUsers(user.id)" type="checkbox">
                                            </div>
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    <div v-if="chats" class="col-md-6 col-lg-5 col-xl-4 mb-4 mb-md-0">
                        <h5 class="font-weight-bold mb-3 text-center text-lg-start">Chats</h5>
                        <div class="card">
                            <div class="card-body">
                                <ul class="list-unstyled mb-0">
                                    <li v-for="chat in chats" class="p-2 border-bottom" style="background-color: #eee;">
                                        <Link :href="route('chat.show', chat.id)">
                                            <a href="#" class="d-flex justify-content-between">
                                                <div class="d-flex flex-row">
                                                    <img src="https://mdbcdn.b-cdn.net/img/Photos/Avatars/avatar-8.webp" alt="avatar"
                                                         class="rounded-circle d-flex align-self-center me-3 shadow-1-strong" width="60">
                                                    <div class="pt-1">
                                                        <p class="fw-bold mb-0">{{chat.title}}</p>
                                                        <p class="small text-muted">{{ chat.last_message.user_name }}: {{chat.last_message.body}}</p>
                                                    </div>
                                                </div>
                                                <div class="pt-1">
                                                    <p class="small text-muted mb-1">{{chat.last_message.time}}</p>
                                                    <span v-if="chat.unreadable_count !== 0" class="badge bg-danger float-end">{{chat.unreadable_count}}</span>
                                                </div>
                                            </a>
                                        </Link>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </AuthenticatedLayout>
</template>

<script>
import {Link} from "@inertiajs/vue3";
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";

export default {
    name: "Index",

    props: [
        'users',
        'chats'
    ],

    data() {
        return {
            isGroup: false,
            userIds: [],
            title: null
        }
    },

    created() {
        window.Echo.channel(`users.${this.$page.props.auth.user.id}`)
            .listen('.store-message-status', res => {
                this.chats.filter(chat => {
                    if (chat.id === res.chat_id) {
                        chat.unreadable_count = res.count
                        chat.last_message = res.message
                    }
                })

            })
    },

    components: {
        AuthenticatedLayout,
        Link
    },


    methods: {
        store(id) {
            this.$inertia.post('/chats', {title: null, users: [id]})
        },

        storeGroup() {
            if (this.userIds.length < 2) return
            this.$inertia.post('/chats', {title: this.title, users: this.userIds})
        },

        toggleUsers(id) {
            let index = this.userIds.indexOf(id)
            if (index === -1) {
                this.userIds.push(id)
            } else {
                this.userIds.splice(index, 1)
            }
        },

        refreshUserIds() {
            this.userIds = []
            this.isGroup = false
        }
    }

}
</script>

<style scoped>

</style>
