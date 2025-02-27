<template>
    <div>
        <el-drawer v-model="drawerVisible" :destroy-on-close="true" :close-on-click-modal="false" size="50%">
            <template #header>
                <DrawerHeader :header="$t('setting.diskClean')" :back="handleClose" />
            </template>
            <div v-loading="loading">
                <el-row type="flex" justify="center">
                    <el-col :span="22">
                        <div v-if="scanStatus !== 'scanned'">
                            <div v-if="scanStatus === 'beforeScan'">
                                <el-text class="clean_title" type="success">
                                    {{ $t('clean.lastCleanTime', [form.lastCleanTime || '-']) }}
                                </el-text>
                                <div class="mt-4">
                                    <el-text type="success">
                                        {{
                                            $t('clean.lastCleanHelper', [
                                                form.lastCleanData || '-',
                                                form.lastCleanSize ? computeSize(Number(form.lastCleanSize)) : '-',
                                            ])
                                        }}
                                    </el-text>
                                </div>
                                <div class="large_button">
                                    <el-button type="primary" size="large" @click="scanData">
                                        {{ $t('clean.scan') }}
                                    </el-button>
                                </div>
                            </div>
                            <div v-if="scanStatus === 'afterScan'">
                                <el-text class="clean_title" type="success">{{ $t('clean.cleanSuccessful') }}</el-text>
                                <div class="mt-4">
                                    <el-text type="success">
                                        {{
                                            $t('clean.currentCleanHelper', [
                                                form.lastCleanData,
                                                computeSize(Number(form.lastCleanSize)),
                                            ])
                                        }}
                                    </el-text>
                                </div>
                                <div class="large_button">
                                    <el-button type="success" size="large" @click="scanData">
                                        {{ $t('clean.reScan') }}
                                    </el-button>
                                </div>
                            </div>
                            <div class="app-card">
                                <el-card class="e-card">
                                    <el-row>
                                        <el-col :span="4">
                                            <el-button icon="Setting" link class="card_icon" />
                                        </el-col>
                                        <el-col :span="20">
                                            <div>
                                                <el-text class="mx-1 card_title" type="primary">
                                                    {{ $t('clean.system') }}
                                                </el-text>
                                            </div>
                                            <span class="input-help">{{ $t('clean.systemHelper') }}</span>
                                        </el-col>
                                    </el-row>
                                </el-card>
                                <el-card class="e-card">
                                    <el-row>
                                        <el-col :span="4">
                                            <el-button icon="Upload" link class="card_icon" />
                                        </el-col>
                                        <el-col :span="20">
                                            <div>
                                                <el-text class="mx-1 card_title" type="primary">
                                                    {{ $t('clean.upload') }}
                                                </el-text>
                                            </div>
                                            <span class="input-help">{{ $t('clean.uploadHelper') }}</span>
                                        </el-col>
                                    </el-row>
                                </el-card>
                                <el-card class="e-card">
                                    <el-row>
                                        <el-col :span="4">
                                            <el-button icon="Download" link class="card_icon" />
                                        </el-col>
                                        <el-col :span="20">
                                            <div>
                                                <el-text class="mx-1 card_title" type="primary">
                                                    {{ $t('clean.download') }}
                                                </el-text>
                                            </div>
                                            <span class="input-help">{{ $t('clean.downloadHelper') }}</span>
                                        </el-col>
                                    </el-row>
                                </el-card>
                                <el-card class="e-card">
                                    <el-row>
                                        <el-col :span="4">
                                            <el-button icon="Document" link class="card_icon" />
                                        </el-col>
                                        <el-col :span="20">
                                            <div>
                                                <el-text class="mx-1 card_title" type="primary">
                                                    {{ $t('clean.systemLog') }}
                                                </el-text>
                                            </div>
                                            <span class="input-help">
                                                {{ $t('clean.systemLogHelper') }}
                                            </span>
                                        </el-col>
                                    </el-row>
                                </el-card>
                            </div>
                        </div>
                        <div v-if="scanStatus === 'scanned'">
                            <div>
                                <el-text class="clean_title" type="primary">
                                    {{ $t('clean.totalScan') }} {{ computeSize(totalSize) }}
                                </el-text>
                                <div class="mt-4">
                                    <el-text type="info">
                                        {{ $t('clean.selectScan') }} {{ computeSize(selectSize) }}
                                    </el-text>
                                </div>
                                <div class="large_button">
                                    <el-button type="primary" size="large" @click="onSubmitClean">
                                        {{ $t('clean.clean') }}
                                    </el-button>
                                </div>
                            </div>
                            <el-scrollbar class="mt-4" max-height="calc(100vh - 235px)">
                                <el-collapse v-model="activeNames">
                                    <el-collapse-item :title="$t('clean.system')" name="system">
                                        <el-tree
                                            ref="systemRef"
                                            :data="cleanData.systemClean"
                                            node-key="id"
                                            :default-checked-keys="systemDefaultCheck"
                                            show-checkbox
                                            :props="defaultProps"
                                            @check-change="onChange"
                                        >
                                            <template #default="{ node, data }">
                                                <div class="float-left">
                                                    <span>{{ load18n(data.label) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span v-if="data.size">{{ computeSize(data.size) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span>{{ loadTag(node, data) }}</span>
                                                </div>
                                            </template>
                                        </el-tree>
                                    </el-collapse-item>
                                    <el-collapse-item :title="$t('clean.upload')" name="upload">
                                        <el-tree
                                            ref="uploadRef"
                                            :data="cleanData.uploadClean"
                                            node-key="id"
                                            :default-checked-keys="uploadDefaultCheck"
                                            show-checkbox
                                            :props="defaultProps"
                                            @check-change="onChange"
                                        >
                                            <template #default="{ node, data }">
                                                <div class="float-left">
                                                    <span>{{ load18n(data.label) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span v-if="data.size">{{ computeSize(data.size) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span>{{ loadTag(node, data) }}</span>
                                                </div>
                                            </template>
                                        </el-tree>
                                    </el-collapse-item>
                                    <el-collapse-item :title="$t('clean.download')" name="download">
                                        <el-tree
                                            ref="downloadRef"
                                            :data="cleanData.downloadClean"
                                            node-key="id"
                                            :default-checked-keys="downloadDefaultCheck"
                                            show-checkbox
                                            :props="defaultProps"
                                            @check-change="onChange"
                                        >
                                            <template #default="{ node, data }">
                                                <div class="float-left">
                                                    <span>{{ load18n(data.label) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span v-if="data.size">{{ computeSize(data.size) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span>{{ loadTag(node, data) }}</span>
                                                </div>
                                            </template>
                                        </el-tree>
                                    </el-collapse-item>
                                    <el-collapse-item :title="$t('clean.systemLog')" name="system_log">
                                        <el-tree
                                            ref="systemLogRef"
                                            :data="cleanData.systemLogClean"
                                            node-key="id"
                                            :default-checked-keys="systemLogDefaultCheck"
                                            show-checkbox
                                            :props="defaultProps"
                                            @check-change="onChange"
                                        >
                                            <template #default="{ node, data }">
                                                <div class="float-left">
                                                    <span>{{ load18n(data.label) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span v-if="data.size">{{ computeSize(data.size) }}</span>
                                                </div>
                                                <div class="ml-4 float-left">
                                                    <span>{{ loadTag(node, data) }}</span>
                                                </div>
                                            </template>
                                        </el-tree>
                                    </el-collapse-item>
                                </el-collapse>
                            </el-scrollbar>
                        </div>
                    </el-col>
                </el-row>
            </div>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="onCancel">{{ $t('commons.button.cancel') }}</el-button>
                </span>
            </template>
        </el-drawer>
    </div>
</template>
<script lang="ts" setup>
import { reactive, ref } from 'vue';
import { computeSize } from '@/utils/util';
import DrawerHeader from '@/components/drawer-header/index.vue';
import { cleanSystem, scanSystem } from '@/api/modules/setting';
import i18n from '@/lang';
import { MsgSuccess } from '@/utils/message';
import { GlobalStore } from '@/store';
const globalStore = GlobalStore();

const emit = defineEmits<{ (e: 'search'): void }>();
const drawerVisible = ref();
const loading = ref();
const totalSize = ref<number>(0);
const selectSize = ref<number>(0);

const scanStatus = ref<string>('beforeScan');

const defaultProps = {
    children: 'children',
    label: 'label',
};
const cleanData = reactive({
    systemClean: [],
    uploadClean: [],
    downloadClean: [],
    systemLogClean: [],
});
const systemRef = ref();
const systemDefaultCheck = ref([]);
const uploadRef = ref();
const uploadDefaultCheck = ref([]);
const downloadRef = ref();
const downloadDefaultCheck = ref([]);
const systemLogRef = ref();
const systemLogDefaultCheck = ref([]);
const activeNames = ref(['system', 'upload', 'download', 'system_log']);

const submitCleans = ref();

const form = reactive({
    lastCleanTime: '',
    lastCleanSize: '',
    lastCleanData: '',
});
interface DialogProps {
    lastCleanTime: string;
    lastCleanSize: string;
    lastCleanData: string;
}
const acceptParams = (params: DialogProps): void => {
    scanStatus.value = 'beforeScan';
    form.lastCleanTime = params.lastCleanTime;
    form.lastCleanSize = params.lastCleanSize;
    form.lastCleanData = params.lastCleanData;
    drawerVisible.value = true;
};

const scanData = async () => {
    loading.value = true;
    await scanSystem()
        .then((res) => {
            loading.value = false;
            selectSize.value = 0;
            totalSize.value = 0;
            cleanData.systemClean = res.data.systemClean || [];
            for (const item of cleanData.systemClean) {
                totalSize.value += item.size;
            }
            cleanData.uploadClean = res.data.uploadClean || [];
            for (const item of cleanData.uploadClean) {
                totalSize.value += item.size;
            }
            cleanData.downloadClean = res.data.downloadClean || [];
            for (const item of cleanData.downloadClean) {
                totalSize.value += item.size;
            }
            cleanData.systemLogClean = res.data.systemLogClean || [];
            for (const item of cleanData.systemLogClean) {
                totalSize.value += item.size;
            }
            loadCheck(cleanData.systemClean, systemDefaultCheck.value);
            loadCheck(cleanData.uploadClean, uploadDefaultCheck.value);
            loadCheck(cleanData.downloadClean, downloadDefaultCheck.value);
            loadCheck(cleanData.systemLogClean, systemLogDefaultCheck.value);
            scanStatus.value = 'scanned';
        })
        .catch(() => {
            loading.value = false;
        });
};

const onSubmitClean = async () => {
    ElMessageBox.confirm(i18n.global.t('clean.cleanHelper'), i18n.global.t('clean.clean'), {
        confirmButtonText: i18n.global.t('commons.button.confirm'),
        cancelButtonText: i18n.global.t('commons.button.cancel'),
        type: 'info',
    }).then(async () => {
        loading.value = true;
        submitCleans.value = [];
        let restart = false;
        loadSubmitCheck(cleanData.systemClean);
        loadSubmitCheck(cleanData.uploadClean);
        loadSubmitCheck(cleanData.downloadClean);
        loadSubmitCheck(cleanData.systemLogClean);
        for (const item of submitCleans.value) {
            if (item.treeType === 'cache') {
                restart = true;
                break;
            }
        }
        await cleanSystem(submitCleans.value)
            .then(() => {
                form.lastCleanSize = selectSize.value + '';
                form.lastCleanData = submitCleans.value.length + '';
                scanStatus.value = 'afterScan';
                MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
                loading.value = false;
                if (restart) {
                    let href = window.location.href;
                    globalStore.isLogin = false;
                    let address = href.split('://')[1];
                    if (globalStore.entrance) {
                        address = address.replaceAll('settings/panel', globalStore.entrance);
                    } else {
                        address = address.replaceAll('settings/panel', 'login');
                    }
                    window.open(`http://${address}`, '_self');
                }
            })
            .catch(() => {
                loading.value = false;
            });
    });
};

const onCancel = async () => {
    drawerVisible.value = false;
    emit('search');
};

const loadSubmitCheck = (data: any) => {
    if (data.children === null) {
        if (data.isCheck) {
            submitCleans.value.push({ treeType: data.type, name: data.name, size: data.size });
        }
        return;
    }
    for (const item of data) {
        if (item.isCheck) {
            submitCleans.value.push({ treeType: item.type, name: item.name, size: item.size });
            continue;
        }
        if (item.children) {
            loadSubmitCheck(item.children);
        }
    }
};

const changeCheckStatus = (data: any, isCheck: boolean) => {
    data.isCheck = isCheck;
    if (data.children) {
        for (const item of data.children) {
            changeCheckStatus(item, isCheck);
        }
    }
};

function onChange(data: any, isCheck: boolean) {
    changeCheckStatus(data, isCheck);
    selectSize.value = 0;
    let systemSelects = systemRef.value.getCheckedNodes(false, true);
    for (const item of systemSelects) {
        if (item.children === null) {
            selectSize.value = selectSize.value + Number(item.size);
        }
    }
    let uploadSelects = uploadRef.value.getCheckedNodes(false, true);
    for (const item of uploadSelects) {
        if (item.children === null) {
            selectSize.value = selectSize.value + Number(item.size);
        }
    }
    let downloadSelects = downloadRef.value.getCheckedNodes(false, true);
    for (const item of downloadSelects) {
        if (item.children === null) {
            selectSize.value = selectSize.value + Number(item.size);
        }
    }
    let systemLogSelects = systemLogRef.value.getCheckedNodes(false, true);
    for (const item of systemLogSelects) {
        if (item.children === null) {
            selectSize.value = selectSize.value + Number(item.size);
        }
    }
}

function loadCheck(data: any, checkList: any) {
    if (data.children === null) {
        if (data.isCheck) {
            checkList.push(data.id);
        }
        return;
    }
    for (const item of data) {
        if (item.isCheck) {
            selectSize.value = selectSize.value + Number(item.size);
            checkList.push(item.id);
            continue;
        }
        if (item.children) {
            loadCheck(item.children, checkList);
        }
    }
}

function loadTag(node: any, data: any) {
    if (node.level !== 1) {
        return '';
    }
    if (data.size === 0) {
        return i18n.global.t('clean.statusClean');
    }
    if (data.label === 'upgrade') {
        return i18n.global.t('clean.upgradeHelper');
    }
    if (data.label === 'cache') {
        return i18n.global.t('clean.cacheHelper');
    }
    return data.isRecommend ? i18n.global.t('clean.statusSuggest') : i18n.global.t('clean.statusWarning');
}

function load18n(label: string) {
    switch (label) {
        case '1panel_original':
            return i18n.global.t('clean.panelOriginal');
        case 'upgrade':
            return i18n.global.t('clean.upgrade');
        case 'cache':
            return i18n.global.t('clean.cache');
        case 'snapshot':
            return i18n.global.t('clean.snapshot');
        case 'snapshot_tmp':
            return i18n.global.t('clean.snapshotTmp');
        case 'snapshot_local':
            return i18n.global.t('clean.snapshotLocal');
        case 'rollback':
            return i18n.global.t('clean.rollback');
        case 'unused':
            return i18n.global.t('clean.unused');
        case 'old_original':
            return i18n.global.t('clean.oldOriginal');
        case 'old_apps_bak':
            return i18n.global.t('clean.oldAppsBak');
        case 'old_upgrade':
            return i18n.global.t('clean.oldUpgrade');
        case 'upload':
        case 'upload_tmp':
            return i18n.global.t('clean.upload');
        case 'download':
            return i18n.global.t('clean.download');
        case 'upload_website':
        case 'rollback_website':
        case 'download_website':
            return i18n.global.t('clean.website');
        case 'upload_app':
        case 'rollback_app':
        case 'download_app':
            return i18n.global.t('clean.app');
        case 'upload_database':
        case 'rollback_database':
        case 'download_database':
            return i18n.global.t('clean.database');
        case 'upload_directory':
        case 'download_directory':
            return i18n.global.t('clean.directory');
        case 'system_log':
            return i18n.global.t('clean.systemLog');
        case 'docker_log':
            return i18n.global.t('clean.dockerLog');
        case 'task_log':
            return i18n.global.t('clean.taskLog');
        case 'shell':
            return i18n.global.t('clean.shell');
        case 'containerShell':
            return i18n.global.t('clean.containerShell');
        case 'curl':
            return i18n.global.t('clean.curl');
        default:
            return label;
    }
}

const handleClose = () => {
    emit('search');
    drawerVisible.value = false;
};

defineExpose({
    acceptParams,
});
</script>

<style lang="scss" scoped>
.app-card {
    cursor: pointer;
    width: 100%;
    &:hover .app-icon {
        transform: scale(1.2);
    }
    .e-card {
        margin-top: 30px;
        cursor: pointer;
        border: var(--panel-border) !important;
        &:hover {
            cursor: pointer;
            border: 1px solid var(--el-color-primary) !important;
        }
    }
}
.card_icon {
    font-size: 36px;
    float: right;
    margin-right: 15px;
}
.card_title {
    font-size: 18px;
}
.clean_title {
    font-size: 22px;
}
.large_button {
    float: right;
    margin-top: -40px;
}
</style>
