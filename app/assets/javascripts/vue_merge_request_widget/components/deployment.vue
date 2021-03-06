<script>
import Icon from '~/vue_shared/components/icon.vue';
import TooltipOnTruncate from '~/vue_shared/components/tooltip_on_truncate.vue';
import FilteredSearchDropdown from '~/vue_shared/components/filtered_search_dropdown.vue';
import { __ } from '~/locale';
import timeagoMixin from '../../vue_shared/mixins/timeago';
import tooltip from '../../vue_shared/directives/tooltip';
import LoadingButton from '../../vue_shared/components/loading_button.vue';
import { visitUrl } from '../../lib/utils/url_utility';
import createFlash from '../../flash';
import MemoryUsage from './memory_usage.vue';
import StatusIcon from './mr_widget_status_icon.vue';
import ReviewAppLink from './review_app_link.vue';
import MRWidgetService from '../services/mr_widget_service';

export default {
  name: 'Deployment',
  components: {
    LoadingButton,
    MemoryUsage,
    StatusIcon,
    Icon,
    TooltipOnTruncate,
    FilteredSearchDropdown,
    ReviewAppLink,
  },
  directives: {
    tooltip,
  },
  mixins: [timeagoMixin],
  props: {
    deployment: {
      type: Object,
      required: true,
    },
    showMetrics: {
      type: Boolean,
      required: true,
    },
  },
  deployedTextMap: {
    running: __('Deploying to'),
    success: __('Deployed to'),
    failed: __('Failed to deploy to'),
  },
  data() {
    return {
      isStopping: false,
    };
  },
  computed: {
    deployTimeago() {
      return this.timeFormated(this.deployment.deployed_at);
    },
    hasExternalUrls() {
      return !!(this.deployment.external_url && this.deployment.external_url_formatted);
    },
    hasDeploymentTime() {
      return !!(this.deployment.deployed_at && this.deployment.deployed_at_formatted);
    },
    hasDeploymentMeta() {
      return !!(this.deployment.url && this.deployment.name);
    },
    hasMetrics() {
      return !!this.deployment.metrics_url;
    },
    deployedText() {
      return this.$options.deployedTextMap[this.deployment.status];
    },
    isDeployInProgress() {
      return this.deployment.status === 'running';
    },
    deployInProgressTooltip() {
      return this.isDeployInProgress
        ? __('Stopping this environment is currently not possible as a deployment is in progress')
        : '';
    },
    shouldRenderDropdown() {
      return this.deployment.changes && this.deployment.changes.length > 0;
    },
    showMemoryUsage() {
      return this.hasMetrics && this.showMetrics;
    },
  },
  methods: {
    stopEnvironment() {
      const msg = __('Are you sure you want to stop this environment?');
      const isConfirmed = confirm(msg); // eslint-disable-line

      if (isConfirmed) {
        this.isStopping = true;

        MRWidgetService.stopEnvironment(this.deployment.stop_url)
          .then(res => res.data)
          .then(data => {
            if (data.redirect_url) {
              visitUrl(data.redirect_url);
            }

            this.isStopping = false;
          })
          .catch(() => {
            createFlash('Something went wrong while stopping this environment. Please try again.');
            this.isStopping = false;
          });
      }
    },
  },
};
</script>

<template>
  <div class="mr-widget-heading deploy-heading append-bottom-default">
    <div class="ci-widget media">
      <div class="media-body">
        <div class="deploy-body">
          <div class="js-deployment-info deployment-info">
            <template v-if="hasDeploymentMeta">
              <span>
                {{ deployedText }}
              </span>
              <tooltip-on-truncate
                :title="deployment.name"
                truncate-target="child"
                class="deploy-link label-truncate"
              >
                <a
                  :href="deployment.url"
                  target="_blank"
                  rel="noopener noreferrer nofollow"
                  class="js-deploy-meta"
                >
                  {{ deployment.name }}
                </a>
              </tooltip-on-truncate>
            </template>
            <span
              v-if="hasDeploymentTime"
              v-tooltip
              :title="deployment.deployed_at_formatted"
              class="js-deploy-time"
            >
              {{ deployTimeago }}
            </span>
            <memory-usage
              v-if="showMemoryUsage"
              :metrics-url="deployment.metrics_url"
              :metrics-monitoring-url="deployment.metrics_monitoring_url"
            />
          </div>
          <div>
            <template v-if="hasExternalUrls">
              <filtered-search-dropdown
                v-if="shouldRenderDropdown"
                class="js-mr-wigdet-deployment-dropdown inline"
                :items="deployment.changes"
                :main-action-link="deployment.external_url"
                filter-key="path"
              >
                <template
                  slot="mainAction"
                  slot-scope="slotProps"
                >
                  <review-app-link
                    :link="deployment.external_url"
                    :css-class="`deploy-link js-deploy-url inline ${slotProps.className}`"
                  />
                </template>

                <template
                  slot="result"
                  slot-scope="slotProps"
                >
                  <a
                    :href="slotProps.result.external_url"
                    target="_blank"
                    rel="noopener noreferrer nofollow"
                    class="menu-item"
                  >
                    <strong class="str-truncated-100 append-bottom-0 d-block">
                      {{ slotProps.result.path }}
                    </strong>

                    <p class="text-secondary str-truncated-100 append-bottom-0 d-block">
                      {{ slotProps.result.external_url }}
                    </p>
                  </a>
                </template>
              </filtered-search-dropdown>
              <review-app-link
                v-else
                :link="deployment.external_url"
                css-class="js-deploy-url js-deploy-url-feature-flag deploy-link btn btn-default btn-sm inlin"
              />
            </template>
            <span 
              v-if="deployment.stop_url"
              v-tooltip 
              :title="deployInProgressTooltip"
              class="d-inline-block" 
              tabindex="0"
            >
              <loading-button
                :loading="isStopping"
                :disabled="isDeployInProgress"
                :title="__('Stop environment')"
                container-class="js-stop-env btn btn-default btn-sm inline prepend-left-4"
                @click="stopEnvironment"
              >
                <icon name="stop" />
              </loading-button>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
