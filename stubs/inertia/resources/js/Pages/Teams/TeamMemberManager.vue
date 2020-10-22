<template>
    <div>
        <div v-if="userPermissions.canAddTeamMembers">
            <jet-section-border />

            <!-- Add Team Member -->
            <jet-form-section @submitted="addTeamMember">
                <template #title>
                    Add Team Member
                </template>

                <template #description>
                    Add a new team member to your team, allowing them to collaborate with you.
                </template>

                <template #form>
                    <div class="mb-3">
                        Please provide the email address of the person you would like to add to this team. The email address must be associated with an existing account.
                    </div>

                    <!-- Member Email -->
                    <div class="mb-2 w-75">
                        <jet-label for="email" value="Email" />
                        <jet-input id="name" type="text" v-model="addTeamMemberForm.email"
                                   :class="{ 'is-invalid': addTeamMemberForm.error('email') }" />
                        <jet-input-error :message="addTeamMemberForm.error('email')" />
                    </div>

                    <!-- Role -->
                    <div class="my-3 w-75" v-if="availableRoles.length > 0">
                        <jet-label for="roles" value="Role" />

                        <input type="hidden" :class="{ 'is-invalid': addTeamMemberForm.error('role') }">
                        <jet-input-error :message="addTeamMemberForm.error('role')" />

                        <div class="list-group">
                            <a href="#" class="list-group-item list-group-item-action" :class="{'text-black-50': addTeamMemberForm.role && addTeamMemberForm.role != role.key}"
                               @click.prevent="addTeamMemberForm.role = role.key"
                               v-for="(role, i) in availableRoles">
                                <div>
                                    <span :class="{'font-weight-bold': addTeamMemberForm.role == role.key}">
                                        {{ role.name }}
                                    </span>

                                    <svg v-if="addTeamMemberForm.role == role.key" class="ml-1 text-success font-weight-light" width="20" fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" stroke="currentColor" viewBox="0 0 24 24"><path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                                </div>

                                <!-- Role Description -->
                                <div class="mt-2">
                                    {{ role.description }}
                                </div>
                            </a>
                        </div>
                    </div>
                </template>

                <template #actions>
                    <jet-action-message :on="addTeamMemberForm.recentlySuccessful" class="mr-3">
                        Added.
                    </jet-action-message>

                    <jet-button :class="{ 'text-white-50': addTeamMemberForm.processing }" :disabled="addTeamMemberForm.processing">
                        Add
                    </jet-button>
                </template>
            </jet-form-section>
        </div>

        <div v-if="team.users.length > 0">
            <jet-section-border />

            <!-- Manage Team Members -->
            <jet-action-section>
                <template #title>
                    Team Members
                </template>

                <template #description>
                    All of the people that are part of this team.
                </template>

                <!-- Team Member List -->
                <template #content>
                    <div class="d-flex justify-content-between mb-3" v-for="user in team.users">
                        <div class="d-flex justify-content-start">
                            <div class="pr-3">
                                <img width="32" class="rounded-circle" :src="user.profile_photo_url" :alt="user.name">
                            </div>
                            <span>{{ user.name }}</span>
                        </div>

                        <div class="d-flex">
                            <!-- Manage Team Member Role -->
                            <button class="btn btn-link text-secondary"
                                    v-if="userPermissions.canAddTeamMembers && availableRoles.length > 0"
                                    @click="manageRole(user)">
                                {{ displayableRole(user.membership.role) }}
                            </button>

                            <div class="btn btn-link text-secondary disabled text-decoration-none ml-2" v-else-if="availableRoles.length > 0">
                                {{ displayableRole(user.membership.role) }}
                            </div>

                            <!-- Leave Team -->
                            <button class="btn btn-link text-danger text-decoration-none"
                                    @click="confirmLeavingTeam"
                                    v-if="$page.user.id === user.id">
                                Leave
                            </button>

                            <!-- Remove Team Member -->
                            <button class="btn btn-link text-danger text-decoration-none"
                                    @click="confirmTeamMemberRemoval(user)"
                                    v-if="userPermissions.canRemoveTeamMembers">
                                Remove
                            </button>
                        </div>
                    </div>
                </template>
            </jet-action-section>
        </div>

        <!-- Role Management Modal -->
        <jet-dialog-modal id="currentlyManagingRoleModal">
            <template #title>
                Manage Role
            </template>

            <template #content>
                <div class="list-group" v-if="managingRoleFor">
                    <a href="#" class="list-group-item list-group-item-action" :class="{'text-black-50': updateRoleForm.role && updateRoleForm.role != role.key}"
                       @click.prevent="updateRoleForm.role = role.key" v-for="(role, i) in availableRoles">
                        <div>
                            <span :class="{'font-weight-bold': updateRoleForm.role == role.key}">
                                {{ role.name }}
                            </span>

                            <svg v-if="updateRoleForm.role == role.key" class="ml-1 text-success font-weight-light" width="20" fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" stroke="currentColor" viewBox="0 0 24 24"><path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                        </div>

                        <!-- Role Description -->
                        <div class="mt-2">
                            {{ role.description }}
                        </div>
                    </a>
                </div>
            </template>

            <template #footer>
                <jet-secondary-button data-dismiss="modal">
                    Nevermind
                </jet-secondary-button>

                <jet-button class="ml-2" @click.native="updateRole" :class="{ 'text-black-50': updateRoleForm.processing }" :disabled="updateRoleForm.processing">
                    Save
                </jet-button>
            </template>
        </jet-dialog-modal>

        <!-- Leave Team Confirmation Modal -->
        <jet-confirmation-modal id="confirmingLeavingTeamModal">
            <template #title>
                Leave Team
            </template>

            <template #content>
                Are you sure you would like to leave this team?
            </template>

            <template #footer>
                <jet-secondary-button data-dismiss="modal">
                    Nevermind
                </jet-secondary-button>

                <jet-danger-button class="ml-2" @click.native="leaveTeam" :class="{ 'text-white-50': leaveTeamForm.processing }" :disabled="leaveTeamForm.processing">
                    Leave
                </jet-danger-button>
            </template>
        </jet-confirmation-modal>

        <!-- Remove Team Member Confirmation Modal -->
        <jet-confirmation-modal id="teamMemberBeingRemovedModal">
            <template #title>
                Remove Team Member
            </template>

            <template #content>
                Are you sure you would like to remove this person from the team?
            </template>

            <template #footer>
                <jet-secondary-button data-dismiss="modal">
                    Nevermind
                </jet-secondary-button>

                <jet-danger-button class="ml-2" @click.native="removeTeamMember" :class="{ 'text-white-50': removeTeamMemberForm.processing }" :disabled="removeTeamMemberForm.processing">
                    Remove
                </jet-danger-button>
            </template>
        </jet-confirmation-modal>
    </div>
</template>

<script>
    import JetActionMessage from './../../Jetstream/ActionMessage'
    import JetActionSection from './../../Jetstream/ActionSection'
    import JetButton from './../../Jetstream/Button'
    import JetConfirmationModal from './../../Jetstream/ConfirmationModal'
    import JetDangerButton from './../../Jetstream/DangerButton'
    import JetDialogModal from './../../Jetstream/DialogModal'
    import JetFormSection from './../../Jetstream/FormSection'
    import JetInput from './../../Jetstream/Input'
    import JetInputError from './../../Jetstream/InputError'
    import JetLabel from './../../Jetstream/Label'
    import JetSecondaryButton from './../../Jetstream/SecondaryButton'
    import JetSectionBorder from './../../Jetstream/SectionBorder'

    export default {
        components: {
            JetActionMessage,
            JetActionSection,
            JetButton,
            JetConfirmationModal,
            JetDangerButton,
            JetDialogModal,
            JetFormSection,
            JetInput,
            JetInputError,
            JetLabel,
            JetSecondaryButton,
            JetSectionBorder,
        },

        props: [
            'team',
            'availableRoles',
            'userPermissions'
        ],

        data() {
            return {
                addTeamMemberForm: this.$inertia.form({
                    email: '',
                    role: null,
                }, {
                    bag: 'addTeamMember',
                    resetOnSuccess: true,
                }),

                updateRoleForm: this.$inertia.form({
                    role: null,
                }, {
                    bag: 'updateRole',
                    resetOnSuccess: false,
                }),

                leaveTeamForm: this.$inertia.form({
                    //
                }, {
                    bag: 'leaveTeam',
                }),

                removeTeamMemberForm: this.$inertia.form({
                    //
                }, {
                    bag: 'removeTeamMember',
                }),

                managingRoleFor: null,
                teamMemberBeingRemoved: null,
                modal: null
            }
        },

        methods: {
            addTeamMember() {
                this.addTeamMemberForm.post(route('team-members.store', this.team), {
                    preserveScroll: true
                });
            },

            manageRole(teamMember) {
                this.managingRoleFor = teamMember
                this.updateRoleForm.role = teamMember.membership.role
                this.currentlyManagingRole = true

                this.modal = new Bootstrap.Modal(document.getElementById('currentlyManagingRoleModal'))
                this.modal.toggle()
            },

            updateRole() {
                this.updateRoleForm.put(route('team-members.update', [this.team, this.managingRoleFor]), {
                    preserveScroll: true,
                }).then(() => {
                    this.modal.toggle()
                })
            },

            confirmLeavingTeam() {
                this.modal = new Bootstrap.Modal(document.getElementById('confirmingLeavingTeamModal'))
                this.modal.toggle()
            },

            leaveTeam() {
                this.leaveTeamForm.delete(route('team-members.destroy', [this.team, this.$page.user])).then(() => {
                    this.modal.toggle()
                })
            },

            confirmTeamMemberRemoval(teamMember) {
                this.teamMemberBeingRemoved = teamMember
                this.modal = new Bootstrap.Modal(document.getElementById('teamMemberBeingRemovedModal'))
                this.modal.toggle()
            },

            removeTeamMember() {
                this.removeTeamMemberForm.delete(route('team-members.destroy', [this.team, this.teamMemberBeingRemoved]), {
                    preserveScroll: true,
                    preserveState: true,
                }).then(() => {
                    this.teamMemberBeingRemoved = null
                    this.modal.toggle()
                })
            },

            displayableRole(role) {
                return this.availableRoles.find(r => r.key == role).name
            },
        },
    }
</script>