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
  <div class="line-button-warpper">
    <div class="line-button">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  data: function () {
    return {};
  },
  methods: {}
};
</script>

<style scoped>
/* .line-button-warpper { */
/* margin-bottom: 10px; */
/* } */

.line-button {
  text-align: center;
  background-color: #ebeef3;
  display: block;
  padding: 8px;
  line-height: 1.5;
  transition: all 0.4s;
  cursor: pointer;
  user-select: none;
}

.line-button:hover {
  color: rgb(73, 73, 73);
  background-color: #cfd1d4;
}
</style>
