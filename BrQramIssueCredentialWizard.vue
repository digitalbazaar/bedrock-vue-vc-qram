<template>
  <br-credential-creator-wizard
    :block-next="blockNext"
    :block-back="blockBack"
    :block-finish="blockFinish"
    :subject="subject"
    :issuer="issuer"
    :flow="steps"
    :vocab="vocab"
    :schema-map="schemaMap"
    :templates="templates"
    :welcome="welcome"
    :review="review"
    @back="back($event)"
    @next="next($event)"
    @finish="finish($event)">
    <template #default="{currentStep, stepIndex}">
      <div class="br-qram-step-content">
        <br-qram-presenter
          v-if="stepIndex === 1"
          style="border: 1px solid #333; height: 300px; width: 300px"
          :block-size="10"
          :max-blocks-per-packet="10"
          :data="data"
          :active="true" />
        <q-btn
          v-if="stepIndex === 2"
          @click="$refs.scanner.scan()">
          Scan
        </q-btn>
        <br-qram-scanner
          v-if="stepIndex === 2"
          ref="scanner"
          :max-video-width="300"
          @result="scanResult($event)" />
      </div>
    </template>
  </br-credential-creator-wizard>
</template>

<script>
/*!
  * Copyright (c) 2019 Digital Bazaar, Inc. All rights reserved.
  */
'use strict';

import {BrCredentialCreatorWizard} from 'bedrock-vue-credential-creator-wizard';
import {BrQramPresenter, BrQramScanner} from 'bedrock-vue-qram';

export default {
  name: 'BrQramIssueCredentialWizard',
  components: {BrCredentialCreatorWizard, BrQramPresenter, BrQramScanner},
  props: {
    challenge: {
      type: String,
      default: '',
      required: true
    },
    subject: {
      type: String,
      default: '',
      required: true
    },
    issuer: {
      type: String,
      default: '',
      required: true
    },
    flow: {
      type: Array,
      default: () => [],
      required: true
    },
    vocab: {
      type: Object,
      default: () => ({}),
      required: true
    },
    schemaMap: {
      type: Object,
      default: () => ({}),
      required: true
    },
    templates: {
      type: Array,
      default: () => [],
      required: true
    },
    welcome: {
      type: Object,
      default: () => undefined
    },
    review: {
      type: Object,
      default: () => undefined
    }
  },
  data() {
    return {
      blockNext: false,
      blockBack: false,
      blockFinish: false,
      credentials: [],
      stepIndex: 0,
      runEncoder: false,
      queryResult: null
    };
  },
  computed: {
    data() {
      // TODO: might need to collect VCs in the flow... if so, expose the
      // query on the VC "recipe"? recipe={flow, query, ...}?
      const query = {
        VerifiablePresentation: {
          query: {
            type: 'DIDAuthentication',
            // type: 'QueryByExample',
            // credentialQuery: [{
            //   required: false,
            //   example: {
            //     '@context': 'https://www.w3.org/2018/credentials/v1',
            //     credentialSubject: {
            //       id: ''
            //     }
            //   }
            // }]
            challenge: 'abc'
          }
        }
      };
      // TODO: use minimal cipher to generate x25519 KAK to include in query
      const json = JSON.stringify(query);
      // TODO: use `pako` lib to zip/deflate json
      return new TextEncoder().encode(json);
    },
    steps() {
      const steps = [{
        icon: {
          name: 'fas fa-certificate',
          size: '65px',
          color: 'primary'
        },
        heading: 'Step 1',
        subheading: 'Have the recipient scan the QR code before continuing',
        name: 'Recipient scans query',
        component: 'slot'
      }, {
        icon: {
          name: 'fas fa-id-badge',
          size: '65px',
          color: 'primary'
        },
        heading: 'Step 2',
        subheading: 'Scan the recipient\'s QR code before continuing',
        name: 'Issuer scans query result',
        component: 'slot'
      }];

      // TODO: add custom steps for QRAM scanning

      steps.push(...this.flow);

      return steps;
    }
  },
  methods: {
    async scanResult(event) {
      console.log('scan result received');
      // TODO: set flag to indicate data is being processed, hide
      // scanner, but do not remove from DOM as it might be turned
      // back on and we need its `$ref`

      try {
        const {data} = event.result;
        // TODO: use `pako` to inflate data
        const json = new TextDecoder().decode(data);
        this.queryResult = JSON.parse(json);
        // TODO: show success message
        console.log('query result received');
        this.blockNext = false;
      } catch(e) {
        // TODO: show message to user
        //console.log('bad result, please try again');
        //this.$refs.scanner.scan();
      }
    },
    back({currentStepIndex}) {
      this.blockNext = (currentStepIndex === 3);
    },
    next({currentStepIndex}) {
      this.blockNext = (currentStepIndex === 1);
    },
    finish({credentials}) {
      this.credentials = credentials;
      this.$emit('credentials', {credentials});
    }
  }
};
</script>

<style lang="scss" scoped>

.br-qram-step-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

</style>
