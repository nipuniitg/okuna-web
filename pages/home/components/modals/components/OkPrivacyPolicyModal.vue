<template>
    <div class="is-flex justify-center align-items-center has-height-100-percent has-padding-20">
        <div class="ok-has-background-primary is-semi-rounded is-relative has-height-100-percent">
            <div
                    class="box ok-has-background-primary-highlight is-paddingless ok-privacy-policy-modal has-height-100-percent-mobile"
                    ref="modalBoxContainer">
                <template v-if="!privacyPolicyMd">
                    <ok-loading-indicator></ok-loading-indicator>
                </template>
                <vue-markdown v-else :html="false" :source="privacyPolicyMd" class="ok-has-text-primary-invert content has-padding-20"/>
            </div>
        </div>
    </div>
</template>

<style lang="scss">
    .ok-privacy-policy-modal {
        overflow-x: auto;
        max-height: 700px;
        max-width: 900px;
        width: 100%;
    }
</style>


<script lang="ts">
    import { Component, Prop, Vue } from "nuxt-property-decorator"
    import { CancelableOperation } from "~/lib/CancelableOperation";
    import { okunaContainer } from "~/services/inversify";
    import { TYPES } from "~/services/inversify-types";
    import { IUtilsService } from "~/services/utils/IUtilsService";
    import { IEnvironmentService } from "~/services/environment/IEnvironmentService";
    import { AxiosResponse } from "~/node_modules/axios";
    import { IHttpService } from "~/services/http/IHttpService";
    import OkLoadingIndicator from '~/components/utils/OkLoadingIndicator.vue';
    import VueMarkdown from 'vue-markdown';

    @Component({
        name: "OkPrivacyPolicyModal",
        components: {OkLoadingIndicator, VueMarkdown},
    })
    export default class OkPrivacyPolicyModal extends Vue {

        @Prop({
            type: Function,
            required: false
        }) readonly returnDataSetter: (data: any) => void;

        private utilsService: IUtilsService = okunaContainer.get<IUtilsService>(TYPES.UtilsService);
        private httpService: IHttpService = okunaContainer.get<IHttpService>(TYPES.HttpService);
        private environmentService: IEnvironmentService = okunaContainer.get<IEnvironmentService>(TYPES.EnvironmentService);
        private requestOperation?: CancelableOperation<any>;

        requestInProgress = false;

        privacyPolicyMd = "";

        mounted() {
            this.loadPrivacyPolicy();
        }


        async loadPrivacyPolicy() {
            if (this.requestInProgress) return;
            this.requestInProgress = true;


            try {
                this.requestOperation = CancelableOperation.fromPromise(this.getPrivacyPolicy());
                this.privacyPolicyMd = await this.requestOperation.value;
            } catch (error) {
                this.utilsService.handleErrorWithToast(error);
            } finally {
                this.requestOperation = null;
                this.requestInProgress = false;
            }
        }

        async getPrivacyPolicy(): Promise<string> {

            const response = await this.httpService.get<string, AxiosResponse<string>>(this.environmentService.privacyPolicyMdUrl, {
                isApiRequest: false,
                appendAuthorizationToken: false
            });

            return response.data;
        }

    }
</script>