<template>
  <div class="filters">
    <div class="filter-row">
      <el-container>
        <span class="filter-label adjusted-position" style="display: inline-block; width: 130px;">
          {{ t('kernelVersion') }}
        </span>
        <el-main :style="{ padding: '0', textAlign: 'left' }">
          <div style="display: flex; align-items: center; flex-wrap: wrap;">
            <span class="filter-label-center">{{ t('versionLabel') }}</span>
            <el-checkbox-button
                v-model="filters.kernel.checkAll"
                :indeterminate="filters.kernel.isIndeterminate"
                @change="handleKernelCheckAll"
                style="margin-right: 15px;"
            >
              {{ t('all') }}
            </el-checkbox-button>

            <el-checkbox-group v-model="filters.kernel.selected" @change="handleKernelChange">
              <el-checkbox-button
                  v-for="item in kernelOptions"
                  :key="item"
                  :value="item.version"
              >
                {{ item.version }}
              </el-checkbox-button>
            </el-checkbox-group>
          </div>
          <div style="display: flex; align-items: center; flex-wrap: wrap; margin-top: 8px">
            <span class="filter-label-center">{{ t('versionNumber') }}</span>
            <el-checkbox-button
                v-model="filters.kernels.checkAll"
                :indeterminate="filters.kernels.isIndeterminate"
                @change="handleFilterCheckAll('kernels')"
                style="margin-right: 15px;"
            >
              {{ t('all') }}
            </el-checkbox-button>

            <el-checkbox-group v-model="filters.kernels.selected" @change="handleFilterChange('kernels')">
              <el-checkbox-button
                  v-for="item in kernelVersions"
                  :key="item.version"
                  :value="item.version"
              >
                {{ item.version }}
              </el-checkbox-button>
            </el-checkbox-group>
          </div>
        </el-main>
      </el-container>
    </div>

    <!-- 下面这块就是把原来误用 filters.kernel / filters.kernels 的部分改成 filters.isa -->
    <div class="filter-row">
      <el-container>
        <span class="filter-label adjusted-position" style="display: inline-block; width: 130px;">
          {{ t('isaBaseline') }}
        </span>
        <el-main :style="{ padding: '0', textAlign: 'left' }">
          <div style="display: flex; align-items: center; flex-wrap: wrap;">
            <span class="filter-label-center">{{ t('isaMabi') }}</span>
            <el-checkbox-button
                v-model="filters.isaMabi.checkAll"
                :indeterminate="filters.isaMabi.isIndeterminate"
                @change="handleFilterCheckAll('isaMabi')"
                style="margin-right: 15px;"
            >
              {{ t('all') }}
            </el-checkbox-button>
            <el-checkbox-group v-model="filters.isaMabi.selected" @change="handleFilterChange('isaMabi')">
              <el-checkbox-button
                  v-for="item in props.isaMabi"
                  :key="item"
                  :value="item.profile"
              >
                {{ item.profile }}
              </el-checkbox-button>
            </el-checkbox-group>
          </div>

          <div style="display: flex; align-items: center; flex-wrap: wrap; margin-top: 8px">
            <span class="filter-label-center">{{ t('isaMarch') }}</span>
            <el-checkbox-button
                v-model="filters.isaMarch.checkAll"
                :indeterminate="filters.isaMarch.isIndeterminate"
                @change="handleFilterCheckAll('isaMarch')"
                style="margin-right: 15px;"
            >
              {{ t('all') }}
            </el-checkbox-button>
            <el-checkbox-group v-model="filters.isaMarch.selected" @change="handleFilterChange('isaMarch')">
              <el-dropdown
                v-for="group in groupedIsaMarch"
                :key="group.letter"
                trigger="click"
                :hide-on-click="false"
                class="group-dropdown-wrapper"
              >
                <template #default>
                  <div
                    class="el-checkbox-button group-dropdown-btn"
                    :class="{ 'is-checked': isGroupChecked(group) }"
                  >
                    <div class="el-checkbox-button__inner">
                      {{ group.letter }}
                    </div>
                  </div>
                </template>
                <template #dropdown>
                  <div class="group-dropdown-menu">
                    <el-checkbox-group
                      v-model="filters.isaMarch.selected"
                      @change="handleFilterChange('isaMarch')"
                    >
                      <el-checkbox
                        v-for="item in group.children"
                        :key="item.profile"
                        :value="item.profile"
                      >
                        {{ item.profile }}
                      </el-checkbox>
                    </el-checkbox-group>
                  </div>
                </template>
              </el-dropdown>
            </el-checkbox-group>
          </div>
        </el-main>
      </el-container>
    </div>

    <!-- 其余过滤项保持原来的循环逻辑 -->
    <div class="filter-row" v-for="(filter, key) in otherFilters" :key="key">
      <span class="filter-label">{{ filter.label }}：</span>
      <div style="display: flex; align-items: center; flex-wrap: wrap;">
        <el-checkbox-button
            v-model="filters[key].checkAll"
            :indeterminate="filters[key].isIndeterminate"
            @change="handleFilterCheckAll(key)"
            style="margin-right: 15px;"
        >
          {{ t('all') }}
        </el-checkbox-button>
        <el-checkbox-group v-model="filters[key].selected" @change="handleFilterChange(key)">
          <el-checkbox-button
              v-for="item in filter.options"
              :key="item.id || item"
              :value="item.profile || item.userspace || item"
          >
            {{ item.profile || item.userspace || item }}
          </el-checkbox-button>
        </el-checkbox-group>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useI18n } from "vue-i18n";
import { computed, reactive, ref } from 'vue';

const { t } = useI18n();
const props = defineProps({
  filters: {
    type: Object,
    required: true,
    default: () => ({
      kernel: {},
      kernels: {},
      isaMabi:{},
      isaMarch:{},
      userspace: {},
      installer: {}
    })
  },
  kernelOptions: {
    type: Array,
    default: () => []
  },
  kernelVersions: {
    type: Array,
    default: () => []
  },
  isaMabi: {
    type: Array,
    default: () => []
  },
  isaMarch: {
    type: Array,
    default: () => []
  },
  otherFilters: {
    type: Object,
    default: () => ({
      isa: { label: 'ISA 基线', options: [] },
      userspace: { label: '预装列表', options: [] },
      installer: { label: '引导器', options: [] }
    })
  },
  groupedIsaMarch: {
    type: Array,
    default: () => []
  }
});

const expanded = reactive({});
function toggleExpand(letter) {
  expanded[letter] = !expanded[letter];
}

function isGroupChecked(group) {
  // 判断该分组下是否有 profile 被选中
  return group.children.some(item => props.filters.isaMarch.selected.includes(item.profile));
}

const updateCheckState = (key) => {
  const filter = props.filters[key];
  const allItems =
      key === 'kernel'
          ? props.kernelOptions.map(v => v.version)
          : key === 'kernels'
              ? props.kernelVersions.map(v => v.version)
              : key === 'isaMabi'
                  ? props.isaMabi.map(v => v.profile)
                  : key === 'isaMarch'
                      ? props.isaMarch.map(v => v.profile)
                      : props.otherFilters[key]?.options?.map(v => v.profile || v.userspace || v) || [];

  const checkedCount = filter.selected.length;
  filter.checkAll = checkedCount === allItems.length;
  filter.isIndeterminate = checkedCount > 0 && checkedCount < allItems.length;
};


// kernel 部分：全选逻辑
const handleKernelCheckAll = (val) => {
  props.filters.kernel.selected = val
      ? props.kernelOptions.map(v => v.version)
      : [];
  props.filters.kernel.isIndeterminate = false;
};

const handleKernelChange = (value) => {
  const checkedCount = value.length;
  props.filters.kernel.checkAll = checkedCount === props.kernelOptions.length;
  props.filters.kernel.isIndeterminate = checkedCount > 0 && checkedCount < props.kernelOptions.length;
};

// 通用：其他 key 的全选逻辑
const handleFilterCheckAll = (key) => {
  const filter = props.filters[key];
  let allOptions = [];
  if (key === 'kernel') {
    allOptions = props.kernelOptions.map(v => v.version);
  } else if (key === 'kernels') {
    allOptions = props.kernelVersions.map(v => v.version);
  } else if (key === 'isaMabi') {
    allOptions = props.isaMabi.map(v => v.profile);
  } else if (key === 'isaMarch') {
    allOptions = props.isaMarch.map(v => v.profile);
  }else {
    allOptions = props.otherFilters[key].options.map(v => v.profile || v.userspace || v);
  }
  filter.selected = filter.checkAll ? allOptions : [];
  updateCheckState(key);
};

// 通用：其他 key 的单项变更时更新状态
const handleFilterChange = (key) => {
  updateCheckState(key);
};

const groupedIsaMarch = computed(() => {
  // 1. 先去重
  const unique = Array.from(new Set(props.isaMarch.map(i => i.profile)))
    .map(profile => props.isaMarch.find(i => i.profile === profile));
  // 2. 分组
  const groups = {};
  unique.forEach(item => {
    const first = item.profile[0].toUpperCase();
    if (!groups[first]) groups[first] = [];
    groups[first].push(item);
  });
  // 3. 排序
  const sortedKeys = Object.keys(groups).sort();
  // 4. 每组内排序
  sortedKeys.forEach(k => {
    groups[k].sort((a, b) => a.profile.localeCompare(b.profile, 'en', { sensitivity: 'base' }));
  });
  // 5. 返回分组后的数组
  return sortedKeys.map(letter => ({
    letter,
    children: groups[letter]
  }));
});
</script>

<style scoped>
/* Dark mode specific color for labels */
.dark .filter-label,
.dark .filter-label-center {
  color: #cccccc !important;
  /* Adjusted color for dark mode */
}

:deep(.el-checkbox-button__inner) {
  font-size: 14px;
  font-weight: 400;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
  border-radius: 5px !important;
  background: var(--theme-input) !important;
  color: var(--theme-text) !important;
  /* border: clamp(1px, 0.2vw, 2px) solid var(--theme-border) !important; */
  transition: all 0.2s;
  white-space: nowrap;
  overflow: visible;
  padding: 8px 16px;
  min-width: 80px;
  height: 36px;
  line-height: 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

:deep(.el-checkbox-button) {
  margin-right: 12px;
  margin-bottom: 10px;
  display: inline-flex;
}

:deep(.el-checkbox-group) {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}

:deep(.el-checkbox-button.is-checked .el-checkbox-button__inner) {
  background: var(--theme-hover) !important;
  color: var(--el-color-primary) !important;
  border: clamp(1px, 0.2vw, 2px) solid var(--el-color-primary) !important;
  box-shadow: none;
  font-weight: 500;
}

:deep(.el-checkbox-button__inner:hover) {
  background: var(--theme-hover) !important;
  border-color: var(--el-color-primary) !important;
  color: var(--el-color-primary) !important;
}

:deep(.el-checkbox-button.is-checked .el-checkbox-button__inner:hover) {
  background: var(--theme-hover) !important;
  border-color: var(--el-color-primary) !important;
  color: var(--el-color-primary) !important;
}

.group-dropdown-btn {
  margin-right: 12px;
  min-width: 60px;
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  border-radius: 5px !important;
  background: var(--theme-input) !important;
  color: var(--theme-text) !important;
  /* border: clamp(1px, 0.2vw, 2px) solid var(--theme-border) !important; */
  transition: all 0.2s;
  height: 36px;
  box-sizing: border-box;
}
.group-dropdown-btn .el-checkbox-button__inner {
  font-size: 14px;
  font-weight: 400;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
  border-radius: 5px !important;
  background: var(--theme-input) !important;
  color: var(--theme-text) !important;
  border: clamp(1px, 0.2vw, 2px) solid var(--theme-border) !important;
  transition: all 0.2s;
  white-space: nowrap;
  overflow: visible;
  padding: 8px 16px;
  min-width: 80px;
  height: 36px;
  line-height: 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.group-dropdown-btn.is-checked .el-checkbox-button__inner {
  background: var(--theme-hover) !important;
  color: var(--el-color-primary) !important;
  border: clamp(1px, 0.2vw, 2px) solid var(--el-color-primary) !important;
  font-weight: 500;
}
.group-dropdown-btn:hover .el-checkbox-button__inner {
  background: var(--theme-hover) !important;
  border-color: var(--el-color-primary) !important;
  color: var(--el-color-primary) !important;
}
.group-dropdown-wrapper {
  display: inline-block;
}

.group-dropdown-menu {
  padding: 8px 0;
  min-width: 140px;
}
.group-dropdown-menu .el-checkbox-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 2px;
  width: 100%;
}
.group-dropdown-menu .el-checkbox {
  width: 100%;
  margin: 0;
  padding: 0 16px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
}

@media screen and (max-width: 900px) {
  .filters {
    padding: 0 2vw;
  }

  .filter-title {
    font-size: clamp(14px, 2vw, 16px);
    margin-bottom: 6px;
  }

  .filter-row {
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: clamp(10px, 2vw, 18px);
  }

  :deep(.el-checkbox-button) {
    margin: 0 !important;
    flex: 0 1 auto;
    min-width: calc(50% - 8px);
    max-width: none;
    box-sizing: border-box;
    display: flex !important;
    align-items: center;
    justify-content: center;
  }

  :deep(.el-checkbox-button__inner) {
    font-size: 13px;
    padding: 6px 12px;
    width: 100%;
    min-width: 60px;
    height: 32px;
    line-height: 20px;
    background: var(--theme-input) !important;
    color: var(--theme-text) !important;
    white-space: nowrap;
    overflow: visible;
    border-radius: 7px !important;
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }

  :deep(.el-checkbox-button.is-checked .el-checkbox-button__inner) {
    font-weight: 500;
  }
}

@media (min-width: 769px) {
  :deep(.el-checkbox-button__inner) {
    background: var(--theme-input) !important;
    color: var(--theme-text) !important;
  }
}
</style>
