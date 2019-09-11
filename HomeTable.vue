<template>
    <div>
        <h1 style="font-size:18px; font-weight:bold; color:#333">
            {{title}}
        </h1>
        <table>
            <thead>
                <tr>
                    <td v-for="column in columns" :key="column.label">
                        {{column.label}}
                    </td>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, index) in data" :key="index" :class="index % 2 !== 0 ? 'dark' : null">
                    <td v-for="column in columns" :key="column.label">
                        {{handleTd(column.prop, row)}}
                    </td>
                </tr>
            </tbody>
        </table>
    </div>    
</template>
<script>
export default {
    props: {
        title: {
            type: String,
            default: ''
        },
        data: {
            type: Array,
            default: []
        },
        columns: {
            type: Array,
            default: []
        }
    },

    methods: {
        handleTd(columnProp, row) {
            return row[columnProp]
        }
    }
}
</script>
<style lang="scss" scoped>
table {
    width: 72%;
    margin-left: 9%;
    margin-top: 40px;
    border: 1px solid #363636;
}
thead tr {
    color: #FFF;
    background-color: #f2637b;
}
tbody tr {
    color: #FFF;
    background-color: #ff839a;
}

tr td {
    padding: 10px;
    background-color: '#36cbcb';
}
.dark {
    background-color: #f2637b;
}
</style>