<template>
    <profile-header
        v-bind="pickProps(props, headerPanelProps)"
        :station="profileInfo.station"
    />

    <div
        id="profile"
        class="row row-of-cards"
    >
        <div class="col-lg-7">
            <template v-if="hasStarted">
                <profile-now-playing
                    v-bind="pickProps(props, nowPlayingPanelProps)"
                />

                <profile-schedule
                    :schedule-items="profileInfo.schedule"
                />

                <profile-streams
                    :station="profileInfo.station"
                />
            </template>
            <template v-else>
                <now-playing-not-started-panel />
            </template>

            <profile-public-pages
                v-bind="pickProps(props, {...publicPagesPanelProps,...embedModalProps})"
            />
        </div>

        <div class="col-lg-5">
            <profile-requests
                v-if="stationSupportsRequests"
                v-bind="pickProps(props, requestsPanelProps)"
            />

            <profile-streamers
                v-if="stationSupportsStreamers"
                v-bind="pickProps(props, streamersPanelProps)"
            />

            <template v-if="hasActiveFrontend">
                <profile-frontend
                    v-bind="pickProps(props, frontendPanelProps)"
                    :frontend-running="profileInfo.services.frontend_running"
                />
            </template>

            <template v-if="hasActiveBackend">
                <profile-backend
                    v-bind="pickProps(props, backendPanelProps)"
                    :backend-running="profileInfo.services.backend_running"
                />
            </template>
            <template v-else>
                <profile-backend-none />
            </template>
        </div>
    </div>
</template>

<script setup lang="ts">
import ProfileStreams from './StreamsPanel.vue';
import ProfileHeader from './HeaderPanel.vue';
import ProfileNowPlaying from './NowPlayingPanel.vue';
import ProfileSchedule from './SchedulePanel.vue';
import ProfileRequests from './RequestsPanel.vue';
import ProfileStreamers from './StreamersPanel.vue';
import ProfilePublicPages from './PublicPagesPanel.vue';
import ProfileFrontend from './FrontendPanel.vue';
import ProfileBackendNone from './BackendNonePanel.vue';
import ProfileBackend from './BackendPanel.vue';
import NowPlayingNotStartedPanel from "./NowPlayingNotStartedPanel.vue";
import {BackendAdapter, FrontendAdapter} from '~/entities/RadioAdapters';
import NowPlaying from '~/entities/NowPlaying';
import {computed} from "vue";
import {useAxios} from "~/vendor/axios";
import backendPanelProps from "./backendPanelProps";
import embedModalProps from "./embedModalProps";
import frontendPanelProps from "./frontendPanelProps";
import headerPanelProps from "./headerPanelProps";
import nowPlayingPanelProps from "./nowPlayingPanelProps";
import publicPagesPanelProps from "./publicPagesPanelProps";
import requestsPanelProps from "./requestsPanelProps";
import streamersPanelProps from "./streamersPanelProps";
import {pickProps} from "~/functions/pickProps";
import useAutoRefreshingAsyncState from "~/functions/useAutoRefreshingAsyncState.ts";

const props = defineProps({
    ...backendPanelProps,
    ...embedModalProps,
    ...frontendPanelProps,
    ...headerPanelProps,
    ...nowPlayingPanelProps,
    ...publicPagesPanelProps,
    ...requestsPanelProps,
    ...streamersPanelProps,
    profileApiUri: {
        type: String,
        required: true
    },
    stationSupportsRequests: {
        type: Boolean,
        required: true
    },
    stationSupportsStreamers: {
        type: Boolean,
        required: true
    }
});

const hasActiveFrontend = computed(() => {
    return props.frontendType !== FrontendAdapter.Remote;
});

const hasActiveBackend = computed(() => {
    return props.backendType !== BackendAdapter.None;
});

const {axiosSilent} = useAxios();

const {state: profileInfo} = useAutoRefreshingAsyncState(
    () => axiosSilent.get(props.profileApiUri).then((r) => r.data),
    {
        station: {
            ...NowPlaying.station
        },
        services: {
            backend_running: false,
            frontend_running: false,
            has_started: false,
            needs_restart: false
        },
        schedule: []
    },
    {
        timeout: 15000
    }
);
</script>
