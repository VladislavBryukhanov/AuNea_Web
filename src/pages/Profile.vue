<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
    <v-flex xs12 sm12 md8 offset-md2 lg8 offset-lg2 xl8 offset-xl-2>
        <v-sheet
            elevation="6">
            <v-form
                @submit.prevent="editProfile"
                v-model="isValid">
                <v-container class="settingsHeader">
                    <v-layout>
                        <v-flex xs12 sm5 class="avatar">
                            <label for="selectAvatar">
                                <v-hover>
                                    <v-avatar
                                        slot-scope="{ hover }"
                                        :class="`elevation-${hover ? 8 : 2}`"
                                        size="125"
                                        d-inlinte-flex>
                                        <img :src="myUser.avatarUrl"/>
                                    </v-avatar>
                                </v-hover>
                            </label>
                            <input
                                @change="onAvatarChange"
                                id="selectAvatar"
                                type="file"
                                class="selectAvatar"/>
                        </v-flex>

                        <v-flex xs12 sm7>
                            <v-text-field
                                :counter="20"
                                :rules="rules.loginRule"
                                dark
                                label="Login"
                                v-model="myUser.login">
                            </v-text-field>
                            <v-text-field
                                :counter="20"
                                :rules="rules.nicknameRule"
                                dark
                                label="Nickname"
                                v-model="myUser.nickname">
                            </v-text-field>
                        </v-flex>
                    </v-layout>

                    <v-slide-x-reverse-transition>
                        <v-btn
                            v-show="profileChanged"
                            type="submit"
                            absolute
                            right
                            fab
                            v-model="isValid"
                            :color="this.isValid ? 'primary' : 'error'">
                            <v-icon>close</v-icon>
                            <v-icon>done</v-icon>
                        </v-btn>
                    </v-slide-x-reverse-transition>

                </v-container>

                <v-dialog
                    v-model="dialogOpened"
                    maxWidth="350">
                    <v-card>
                        <v-card-title  class="headline">
                            Maximal file size exceeded
                        </v-card-title>

                        <v-card-text>
                            Maximum image size is 8MB. You have exceeded this restriction.
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn
                                flat
                                color="primary"
                                @click="dialogOpened = !dialogOpened">
                                Ok
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>

                <v-container class="profile">
                    <v-layout row wrap>

                        <v-flex xs12>
                            <v-textarea
                                :counter="400"
                                :rules="rules.bioRule"
                                label="Bio"
                                v-model="myUser.bio">
                            </v-textarea>
                        </v-flex>

                        <v-flex xs12>
                            <v-select
                                label="Theme"
                                :items="themas">
                            </v-select>
                        </v-flex>
                    </v-layout>
                </v-container>
            </v-form>
        </v-sheet>
    </v-flex>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import { Action, State } from 'vuex-class';
import { User } from '../models/User.interface';
import _ from 'lodash';
import fileQuotas from '../constants/fileQuotas';

@Component
export default class Profile extends Vue {

    @State('myAccount', { namespace: 'Auth' })
    private myAccount: User;

    @Action('editProfile', { namespace: 'Auth' })
    private editProfileAction;

    public profileChanged = false;
    public dialogOpened = false;
    public isValid = false;
    public rules = {
        loginRule: [
            (v) => !!v || 'Login is required field',
            (v) => (v.length >= 3 && v.length <= 20) || 'Login must be longer then 3 and less then 20 characters',
        ],
        nicknameRule: [
            (v) => (v.length >= 3 && v.length <= 20) || 'Nickname must be longer then 3 and less then 20 characters',
        ],
        bioRule: [
            (v) => v.length <= 400 || 'Bio must be less then 400 characters',
        ],
    };

    public themas = [
        'Default',
        'Material Light',
        'Material Dark',
    ];

    // this.myAccount is not working (as i understood this.myAccount is async)
    public myUser: User = Object.assign({}, this.$store.state.Auth.myAccount);
    public newAvatar: File;

    @Watch('myUser', { deep: true })
    public myUserChanged() {
        this.profileChanged = !_.isEqual(this.myUser, this.myAccount);
    }

    public async editProfile() {
        if (this.isValid && this.profileChanged) {
            await this.editProfileAction({ changedUser: this.myUser, avatar: this.newAvatar });
            this.$router.back();
        }
    }

    public onAvatarChange(e) {
        const [ avatarFile ] = e.target.files;
        if (avatarFile.size > fileQuotas.MAX_IMAGE_FILE_SIZE) {
            this.dialogOpened = !this.dialogOpened;
            return;
        }

        this.newAvatar = avatarFile;
        const fileReader = new FileReader();
        fileReader.onload = () => {
            this.myUser.avatarUrl = fileReader.result as string;
        };
        fileReader.readAsDataURL(avatarFile);
    }
}
</script>

<style lang="scss" scoped>
    @import "../assets/scss/pages/Profile";
</style>
