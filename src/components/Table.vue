<template>
  <div>
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Group</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="item in items"
          :key="item.index"
        >
          <td>
            <span :style="{color:item.color}">■</span>
            {{ item.name }}
          </td>
          <td class="group_number">
            <span
              v-for="(gj, j) in groups"
              :key="gj.index"
              :style="{ color: j == ingroup[item.index] ? gj.color : '#eeeeee' }"
            >
              {{ j+1 }}
            </span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  props: {
    items: {
      type: Array,
      required: true,
    },
    groups: {
      type: Array,
      required: true,
    },
    ingroup: {
      type: Array,
      required: true,
    },
  },
}
</script>

<style scoped>
table {
  border-collapse: collapse;
}
th {
  background-color: #eeeeee;
}
th, td {
  border: 1px solid #000000;
  padding: 0em 0.5em;
  white-space: nowrap;
}
.group_number {
  color: #eeeeee;
}
</style>
