<!--
  Copyright (C) 2022 Suwings(https://github.com/Suwings)

  This program is free software: you can redistribute it and/or modify
  it under the terms of the GNU General Public License as published by
  the Free Software Foundation, either version 3 of the License, or
  (at your option) any later version.

  This program is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  GNU General Public License for more details.
  
  According to the GPL, it is forbidden to delete all copyright notices, 
  and if you modify the source code, you must open source the
  modified source code.

  版权所有 (C) 2022 Suwings(https://github.com/Suwings)

  本程序为自由软件，你可以依据 GPL 的条款（第三版或者更高），再分发和/或修改它。
  该程序以具有实际用途为目的发布，但是并不包含任何担保，
  也不包含基于特定商用或健康用途的默认担保。具体细节请查看 GPL 协议。

  根据协议，您必须保留所有版权声明，如果修改源码则必须开源修改后的源码。
  前往 https://mcsmanager.com/ 申请闭源开发授权或了解更多。
-->

<template>
  <div class="line-option-list-wrapper" v-if="options[optionKey]">
    <div v-for="(item, index) in options[optionKey]" :key="index">
      <LineOption
        v-if="valueType(options[optionKey][index]) != -1"
        :option-value="options[optionKey]"
        :option-key="index"
      >
        <template #title>
          <span v-if="topTitle">{{ topTitle }}/</span><span>{{ optionKey }}/{{ index }}</span>
        </template>
        <template #info>
          <span v-if="description">{{ description[index] }}</span>
        </template>
      </LineOption>
    </div>
  </div>
</template>

<script>
import LineOption from "./LineOption";

export default {
  components: { LineOption },
  props: ["options", "optionKey", "topTitle", "description"],
  data: function () {
    return {};
  },
  computed: {
    // optionList() {
    //   return this.list;
    // }
  },
  methods: {
    valueType(value) {
      if (typeof value === "object") return -1;
    }
  }
};
</script>
