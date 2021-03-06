<template>
    <v-layout column justify-center>
        <v-expansion-panel>
            <v-expansion-panel-content v-for="item in items" :key="item._id" hide-actions>
                <template #header>
                    <v-layout align-center row spacer>
                        <v-flex sm1 md1 hidden-xs-only>
                            <v-avatar color="primary" size="36px">
                                <img v-if="item.avatarImage" :src="item.avatarImage" alt="Avatar">
                                <span v-else class="white--text headline">{{ item.name | firstLetter }}</span>
                            </v-avatar>
                        </v-flex>

                        <v-flex sm2 md1 hidden-xs-only>
                            <v-icon v-if="item.favorite" color="pink darken-1">favorite</v-icon>
                            <v-icon v-if="item.inDevelopment || item.completed" color="primary">{{ getStateIcon(item) }}</v-icon>
                        </v-flex>

                        <v-flex xs12 sm5 md7>
                            <strong>{{ item.name }} <v-chip small v-if="showCategory(item)" :color="item.category.color" :dark="item.category.dark">{{ item.category.name }}</v-chip></strong>
                        </v-flex>

                        <v-flex md4 sm4 hidden-xs-only style="text-align: right">
                            <v-rating v-model="item.rating" readonly></v-rating>
                        </v-flex>
                    </v-layout>
                </template>

                <v-card>
                    <v-divider></v-divider>
                    <v-card-text>
                        <div style="white-space: pre-line;">{{ item.description }}</div>
                        <div class="pt-2"><v-chip color="primary" small text-color="white" v-for="tag in item.tags" :key="tag">{{tag}}</v-chip></div>
                        <div class="pt-3" v-if="item.www || item.patreon || item.links.length">
                            <div class="mb-2"><v-icon small class="mr-2">share</v-icon><strong>{{ $t('links') }}</strong></div>
                            <div v-if="item.patreon"><v-btn flat small class="text-lowercase my-0" @click="patreonLinkClicked(item.patreon)"><v-icon small class="mr-2">payment</v-icon> https://patreon.com/{{item.patreon}}</v-btn></div>
                            <div v-if="item.www"><v-btn flat small class="text-lowercase my-0" @click="linkClicked(item.www)"><v-icon small class="mr-2">public</v-icon> {{ item.www }}</v-btn></div>
                            <div v-for="link in item.links" :key="link"><v-btn flat small class="text-lowercase my-0" @click="linkClicked(link)"><v-icon small class="mr-2">cloud_queue</v-icon> {{ link }}</v-btn></div>
                        </div>
                    </v-card-text>
                    <v-card-actions>
                        <v-badge color="red" overlap v-if="item.screenshots.length" class="mr-3">
                            <span slot="badge">{{ item.screenshots.length }}</span>
                            <v-btn color="primary" @click="$emit('show-screenshots', item)"><v-icon class="mr-2">photo</v-icon> {{ $t('item.screenshots') }}</v-btn>
                        </v-badge>
                        <v-spacer></v-spacer>
                        <v-tooltip bottom>
                            <template #activator="{ on }">
                                <v-btn color="secondary" @click="editItem(item)" v-on="on" class="mr-2" icon><v-icon>edit</v-icon></v-btn>
                            </template>
                            <span>{{ $t('edit') }}</span>
                        </v-tooltip>
                        <v-tooltip bottom>
                            <template #activator="{ on }">
                                <v-btn color="error" @click="deleteItem(item)" v-on="on" icon><v-icon>delete</v-icon></v-btn>
                            </template>
                            <span>{{ $t('delete') }}</span>
                        </v-tooltip>
                    </v-card-actions>
                </v-card>
            </v-expansion-panel-content>
        </v-expansion-panel>
    </v-layout>
</template>

<script>
    import { shell, clipboard, remote } from 'electron';

    let linksMenu;

    export default {
        name: 'ItemsTable',

        props : ['items', 'filters'],

        data () {
            return {
                /**
                 * Link address that was last clicked.
                 */
                clickedLink : '',
            };
        },

        filters : {
            /**
             * Filter returning only first letter of a string.
             *
             * @param {string} value
             * @returns {string}
             */
            firstLetter (value) {
                if (value) {
                    return value.substr(0, 1);
                }
                return '';
            }
        },

        methods : {
            editItem (item) {
                this.$emit('edit-click', JSON.parse(JSON.stringify(item)));
            },

            deleteItem (item) {
                this.$emit('delete-clicked', item);
            },

            showCategory (item) {
                return item.category && !this.filters.category;
            },

            linkClicked (link) {
                this.clickedLink = link;

                switch (this.$store.getters.settings.linkClickAction) {
                    case 'open':
                        this.openLink();
                        break;
                    case 'copy':
                        this.copyLink();
                        break;
                    case 'ask':
                        linksMenu.popup();
                        break;
                }
            },

            patreonLinkClicked (link) {
                this.clickedLink = 'https://patreon.com/' + link;

                switch (this.$store.getters.settings.patreonClickAction) {
                    case 'open':
                        this.openLink();
                        break;
                    case 'copy':
                        this.copyLink();
                        break;
                    case 'ask':
                        linksMenu.popup();
                        break;
                }
            },

            openLink () {
                shell.openExternal(this.clickedLink);
            },

            copyLink () {
                clipboard.writeText(this.clickedLink);
                this.$emit('show-info', this.$t('linkCopiedToClipboard'));
            },

            getStateIcon (item) {
                if (item.inDevelopment && !item.completed) {
                    return 'build';
                } else if (item.completed && !item.inDevelopment) {
                    return 'done';
                } else {
                    return 'done_outline';
                }
            }
        },

        created () {
            linksMenu = remote.Menu.buildFromTemplate([
                {
                    label : this.$t('settings.linkClickAction.open'),
                    click : this.openLink,
                },
                {
                    label : this.$t('settings.linkClickAction.copy'),
                    click : this.copyLink,
                }
            ]);
        }
    };
</script>

<style scoped>
</style>