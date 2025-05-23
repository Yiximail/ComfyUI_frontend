<template>
  <div>
    <!-- This component does not render anything visible. -->
  </div>
</template>

<script setup lang="ts">
import type { LGraphNode } from '@comfyorg/litegraph'
import { BadgePosition } from '@comfyorg/litegraph'
import { LGraphBadge } from '@comfyorg/litegraph'
import _ from 'lodash'
import { computed, onMounted, watch } from 'vue'

import { app } from '@/scripts/app'
import { ComfyNodeDefImpl, useNodeDefStore } from '@/stores/nodeDefStore'
import { useSettingStore } from '@/stores/settingStore'
import { useColorPaletteStore } from '@/stores/workspace/colorPaletteStore'
import { NodeBadgeMode } from '@/types/nodeSource'

const settingStore = useSettingStore()
const colorPaletteStore = useColorPaletteStore()

const nodeSourceBadgeMode = computed(
  () => settingStore.get('Comfy.NodeBadge.NodeSourceBadgeMode') as NodeBadgeMode
)
const nodeIdBadgeMode = computed(
  () => settingStore.get('Comfy.NodeBadge.NodeIdBadgeMode') as NodeBadgeMode
)
const nodeLifeCycleBadgeMode = computed(
  () =>
    settingStore.get('Comfy.NodeBadge.NodeLifeCycleBadgeMode') as NodeBadgeMode
)

watch([nodeSourceBadgeMode, nodeIdBadgeMode, nodeLifeCycleBadgeMode], () => {
  app.graph?.setDirtyCanvas(true, true)
})

const nodeDefStore = useNodeDefStore()
function badgeTextVisible(
  nodeDef: ComfyNodeDefImpl | null,
  badgeMode: NodeBadgeMode
): boolean {
  return !(
    badgeMode === NodeBadgeMode.None ||
    (nodeDef?.isCoreNode && badgeMode === NodeBadgeMode.HideBuiltIn)
  )
}

onMounted(() => {
  app.registerExtension({
    name: 'Comfy.NodeBadge',
    nodeCreated(node: LGraphNode) {
      node.badgePosition = BadgePosition.TopRight

      const badge = computed(() => {
        const nodeDef = nodeDefStore.fromLGraphNode(node)
        return new LGraphBadge({
          text: _.truncate(
            [
              badgeTextVisible(nodeDef, nodeIdBadgeMode.value)
                ? `#${node.id}`
                : '',
              badgeTextVisible(nodeDef, nodeLifeCycleBadgeMode.value)
                ? nodeDef?.nodeLifeCycleBadgeText ?? ''
                : '',
              badgeTextVisible(nodeDef, nodeSourceBadgeMode.value)
                ? nodeDef?.nodeSource?.badgeText ?? ''
                : ''
            ]
              .filter((s) => s.length > 0)
              .join(' '),
            {
              length: 31
            }
          ),
          fgColor:
            colorPaletteStore.completedActivePalette.colors.litegraph_base
              .BADGE_FG_COLOR,
          bgColor:
            colorPaletteStore.completedActivePalette.colors.litegraph_base
              .BADGE_BG_COLOR
        })
      })

      node.badges.push(() => badge.value)
    }
  })
})
</script>
