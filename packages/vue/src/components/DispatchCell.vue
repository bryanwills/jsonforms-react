<template>
  <component :is="determinedCell" v-bind="cell">
    <!-- Forward all slots dynamically -->
    <template v-for="(_, slotName) in $slots" :key="slotName" #[slotName]>
      <slot :name="slotName"></slot>
    </template>
  </component>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import UnknownRenderer from './UnknownRenderer.vue';
import maxBy from 'lodash/maxBy';
import {
  rendererProps,
  useJsonFormsDispatchCell,
} from '../jsonFormsCompositions';
import { ControlElement } from '@jsonforms/core';

export default defineComponent({
  name: 'DispatchCell',
  props: {
    ...rendererProps<ControlElement>(),
  },
  setup(props) {
    return useJsonFormsDispatchCell(props);
  },
  computed: {
    determinedCell(): any {
      const testerContext = {
        rootSchema: this.cell.rootSchema,
        config: this.config,
      };
      const cell = maxBy(this.cell.cells, (r) =>
        r.tester(this.cell.uischema, this.cell.schema, testerContext)
      );
      if (
        cell === undefined ||
        cell.tester(this.cell.uischema, this.cell.schema, testerContext) === -1
      ) {
        return UnknownRenderer;
      } else {
        return cell.cell;
      }
    },
  },
});
</script>
