<template>
    <q-page class="q-pa-md">
        <q-table :columns="userColumns" :rows="users" :loading="loading" :pagination="pagination">
            <template v-slot:top>
                <q-space/>
                <span>
                    <q-btn icon="add" @click="this.showUserAdd = true"/>
                    <q-tooltip>Add new user</q-tooltip>
                </span>
                <span>
                    <q-btn icon="sync" @click="loadUsers"/>
                    <q-tooltip>Refresh users</q-tooltip>
                </span>
            </template>

            <template v-slot:body-cell-createdAt="props">
                <q-td :props="props">
                    <span>
                        {{ fromNow(props.row.createdAt) }}
                        <q-tooltip class="bg-grey-5 text-black">
                            {{ formatDate(props.row.createdAt) }}
                        </q-tooltip>
                    </span>
                </q-td>
            </template>

            <template v-slot:body-cell-updatedAt="props">
                <q-td :props="props">
                    <span>
                        {{ fromNow(props.row.updatedAt) }}
                        <q-tooltip class="bg-grey-5 text-black">
                            {{ formatDate(props.row.updatedAt) }}
                        </q-tooltip>
                    </span>
                </q-td>
            </template>

            <template v-slot:body-cell-actions="props">
                <q-td :props="props">
                    <q-btn size="sm" icon="delete" @click="deleteUser(props.row.displayName, props.row.id)">
                        <q-tooltip>Delete User</q-tooltip>
                    </q-btn>
                </q-td>
            </template>
        </q-table>

        <!-- TODO: Update this dialog to incorporate third-party user lookups when we have built that capability -->
        <q-dialog v-model="showUserAdd">
            <q-card>
                <q-card-section>
                    <div class="text-h5">Add new user</div>
                </q-card-section>
                <q-card-section>
                    <div class="row q-mb-md">
                        <q-input dense style="min-width: 500px" v-model="user.firstName" label="First Name" @keyup="updateDisplayName" :rules="[val => !!val || 'First Name is required']"/>
                    </div>
                    <div class="row q-mb-md">
                        <q-input dense style="min-width: 500px" v-model="user.lastName" label="Last Name" @keyup="updateDisplayName" :rules="[val => !!val || 'Last Name is required']"/>
                    </div>
                    <div class="row q-mb-md">
                        <q-input dense style="min-width: 500px" v-model="user.displayName" label="Display Name" :rules="[val => !!val || 'Display Name is required']"/>
                    </div>
                    <div class="row q-mb-md">
                        <q-input dense style="min-width: 500px" v-model="user.username" label="Username" :rules="[val => !!val || 'Username is required']"/>
                    </div>
                </q-card-section>
                <q-card-actions align="right">
                    <q-btn flat v-close-popup>Cancel</q-btn>
                    <q-btn flat color="primary" @click="createUser">Save</q-btn>
                </q-card-actions>
            </q-card>
        </q-dialog>
    </q-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { api } from 'boot/axios';
import { notifyError } from 'src/utils/alerts';
import { fromNow, formatDate } from 'src/utils/time';
import { AxiosResponse } from 'axios';

export default defineComponent({
    name: 'UserPage',
    data() {
        return {
            userColumns: [
                {
                    name: 'name',
                    label: 'Name',
                    field: 'displayName',
                    align: 'left'
                },
                {
                    name: 'createdAt',
                    label: 'Created At',
                    align: 'left'
                },
                {
                    name: 'updatedAt',
                    label: 'Updated At',
                    align: 'left'
                },
                {
                    name: 'actions',
                    label: 'Actions',
                    align: 'left'
                }
            ],
            users: [],
            loading: false,
            pagination: {
                rowsPerPage: 20
            },
            showUserAdd: false,
            user: {
                firstName: '',
                lastName: '',
                displayName: '',
                username: ''
            }
        };
    },
    methods: {
        fromNow: fromNow,
        formatDate: formatDate,
        loadUsers() {
            this.loading = true;
            console.log(this.$q);
            api.get('/users')
                .then((response: AxiosResponse) => { this.users = response.data.content; })
                .catch((error: Error) => notifyError('Unable to load users', error))
                .finally(() => { this.loading = false; });
        },
        updateDisplayName() {
            this.user.displayName = `${this.user.firstName} ${this.user.lastName}`;
        },
        createUser() {
            this.showUserAdd = false;

            const userData = {
                firstName: this.user.firstName,
                lastName: this.user.lastName,
                displayName: this.user.displayName,
                systemIdentifier: this.user.username
            };

            api.post('/users', userData)
                .then(() => this.loadUsers())
                .catch((error: Error) => notifyError('Unable to create user', error));
        },
        deleteUser(name: string, id: number) {
            this.$q.dialog({
                title: 'Hold Up!',
                message: `Are you sure you want to delete user ${name}?`,
                cancel: true,
                persistent: true
            }).onOk(() => {
                api.delete(`/users/${id}`)
                    .then(() => this.loadUsers())
                    .catch((error: Error) => notifyError('Failed to delete user', error));
            });
        }
    },
    mounted() {
        this.loadUsers();
    }
});
</script>
