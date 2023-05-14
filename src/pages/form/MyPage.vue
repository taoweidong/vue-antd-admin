<template>
    <!-- @proxy-query="proxyQueryEvent" -->
    <vxe-grid ref='xGrid' header-align="center" @toolbar-button-click="toolbarButtonClickEvent" v-bind="gridOptions"
        @proxy-delete="proxyDeleteEvent" @proxy-save="proxySaveEvent">
        <template #operate="{ row }">
            <vxe-button icon="vxe-icon-edit" title="编辑" circle @click="editRowEvent(row)"></vxe-button>
            <vxe-button icon="vxe-icon-delete" title="删除" circle @click="removeRowEvent(row)"></vxe-button>
        </template>
    </vxe-grid>
</template>

<script>
import { mapState } from 'vuex'
import XEUtils from 'xe-utils'
import VXETable from 'vxe-table'
import { message } from 'ant-design-vue';
import { dataSource as ds } from '@/services'

export default {
    name: 'MyPage',
    data() {
        return {
            gridOptions: {
                border: true,
                showHeaderOverflow: true,
                showOverflow: true,
                keepSource: true,
                id: 'full_edit_1',
                height: 800,
                rowId: 'id',
                rowConfig: {
                    isHover: true
                },
                columnConfig: {
                    resizable: true
                },
                customConfig: {
                    storage: true,
                    checkMethod: this.checkColumnMethod
                },
                printConfig: {
                    columns: [
                        { field: 'name' },
                        { field: 'email' },
                        { field: 'nickname' },
                        { field: 'age' },
                        { field: 'amount' }
                    ]
                },
                sortConfig: {
                    trigger: 'cell',
                    remote: true
                },
                filterConfig: {
                    remote: true
                },
                pagerConfig: {
                    pageSize: 10,
                    pageSizes: [5, 10, 15, 20, 50, 100, 200, 500, 1000]
                },
                formConfig: {
                    titleWidth: 100,
                    titleAlign: 'right',
                    items: [
                        { field: 'name', title: '名称', span: 8, titlePrefix: { message: '名称', icon: 'vxe-icon-info-circle-fill' }, itemRender: { name: '$input', props: { placeholder: '请输入名称' } } },
                        { field: 'email', title: '邮件', span: 8, titlePrefix: { useHTML: true, message: '点击链接：<a class="link" href="https://vxetable.cn" target="_blank">vxe-table官网</a>', icon: 'vxe-icon-question-circle-fill' }, itemRender: { name: '$input', props: { placeholder: '请输入邮件' } } },
                        { field: 'nickname', title: '昵称', span: 8, itemRender: { name: '$input', props: { placeholder: '请输入昵称' } } },
                        { field: 'role', title: '角色', span: 8, folding: true, itemRender: { name: '$input', props: { placeholder: '请输入角色' } } },
                        { field: 'sex', title: '性别', span: 8, folding: true, titleSuffix: { message: '注意，必填信息！', icon: 'vxe-icon-info-circle-fill' }, itemRender: { name: '$select', options: [] } },
                        { field: 'age', title: '年龄', span: 8, folding: true, itemRender: { name: '$input', props: { type: 'number', min: 1, max: 120, placeholder: '请输入年龄' } } },
                        { span: 24, align: 'center', collapseNode: true, itemRender: { name: '$buttons', children: [{ props: { type: 'submit', content: '查询', status: 'primary' } }, { props: { type: 'reset', content: '重置' } }] } }
                    ]
                },
                toolbarConfig: {
                    buttons: [
                        { code: 'insert_button_actived', name: '新增', icon: 'vxe-icon-square-plus' },
                        { code: 'delete_button_actived', name: '直接删除', icon: 'vxe-icon-delete' },
                        { code: 'mark_cancel_button_actived', name: '删除/取消', icon: 'vxe-icon-delete' },
                        { code: 'my_save_button_actived', name: '保存', icon: 'vxe-icon-save', status: 'success' }
                    ],
                    refresh: true,
                    import: false,
                    export: true,
                    print: false,
                    zoom: true,
                    custom: true
                },
                proxyConfig: {
                    seq: true, // 启用动态序号代理，每一页的序号会根据当前页数变化
                    sort: true, // 启用排序代理，当点击排序时会自动触发 query 行为
                    filter: true, // 启用筛选代理，当点击筛选时会自动触发 query 行为
                    form: true, // 启用表单代理，当点击表单提交按钮时会自动触发 reload 行为
                    // 对应响应结果 { result: [], page: { total: 100 } }
                    props: {
                        result: 'result', // 配置响应结果列表字段
                        total: 'page.total' // 配置响应结果总页数字段
                    },
                    // 只接收Promise，具体实现自由发挥
                    ajax: {
                        // 当点击工具栏查询按钮或者手动提交指令 query或reload 时会被触发
                        // query: ({ page, sorts, filters, form }) => {
                        //     const queryParams = Object.assign({}, form)
                        //     // 处理排序条件
                        //     const firstSort = sorts[0]
                        //     if (firstSort) {
                        //         queryParams.sort = firstSort.property
                        //         queryParams.order = firstSort.order
                        //     }
                        //     // 处理筛选条件
                        //     filters.forEach(({ property, values }) => {
                        //         queryParams[property] = values.join(',')
                        //     })
                        //     return fetch(`${this.serveApiUrl}/api/pub/page/list/${page.pageSize}/${page.currentPage}`, queryParams).then(response => response.json())
                        // },
                        query: ({ page, sorts, filters, form }) => {
                            console.log("触发查询。。。。。。。。。。。")

                            const queryParams = Object.assign({}, form)
                            // 处理排序条件
                            const firstSort = sorts[0]
                            if (firstSort) {
                                queryParams.sort = firstSort.property
                                queryParams.order = firstSort.order
                            }
                            // 处理筛选条件
                            filters.forEach(({ property, values }) => {
                                queryParams[property] = values.join(',')
                            })

                            console.log("查询条件：", queryParams)
                            // const currentPage = page.currentPage
                            // const pageSize = page.pageSize

                            queryParams.currentPage = page.currentPage
                            queryParams.pageSize = page.pageSize

                            console.log("接口查詢。。。。开始")
                            return new Promise(resolve => {
                                // setTimeout(() => {
                                //     const list = [
                                //         { id: 10001, name: 'Test1', nickname: 'T1', role: 'Develop', sex: '0', sex2: ['0'], num1: 40, age: 28, address: 'Shenzhen', date12: '', date13: '' },
                                //         { id: 10002, name: 'Test2', nickname: 'T2', role: 'Designer', sex: '1', sex2: ['0', '1'], num1: 44, age: 22, address: 'Guangzhou', date12: '', date13: '2020-08-20' },
                                //         { id: 10003, name: 'Test3', nickname: 'T3', role: 'Test', sex: '0', sex2: ['1'], num1: 200, age: 32, address: 'test abc', date12: '2020-09-10', date13: '' },
                                //         { id: 10004, name: 'Test4', nickname: 'T4', role: 'Designer', sex: '1', sex2: ['1'], num1: 30, age: 23, address: 'Shenzhen', date12: '', date13: '2020-12-04' },
                                //         { id: 10005, name: 'Test5', nickname: 'T5', role: 'Develop', sex: '0', sex2: ['1', '0'], num1: 20, age: 30, address: 'Shanghai', date12: '2020-09-20', date13: '' },
                                //         { id: 10006, name: 'Test6', nickname: 'T6', role: 'Designer', sex: '1', sex2: ['0'], num1: 10, age: 21, address: 'Shenzhen', date12: '', date13: '' },
                                //         { id: 10007, name: 'Test7', nickname: 'T7', role: 'Develop', sex: '0', sex2: ['0'], num1: 5, age: 29, address: 'test abc', date12: '2020-01-02', date13: '2020-09-20' },
                                //         { id: 10008, name: 'Test8', nickname: 'T8', role: 'PM', sex: '1', sex2: ['0'], num1: 2, age: 35, address: 'Shenzhen', date12: '', date13: '' }
                                //     ]
                                //     resolve(list)
                                // }, 500)

                                ds.goodsList({ queryParams }).then(result => {
                                    console.log("接口查詢。。。。完成")
                                    let list = result.data.data.record
                                    console.log(result.data.data)
                                    resolve({
                                        page: {
                                            total: result.data.data.total
                                        },
                                        result: list
                                    })
                                })
                            })


                        },
                        // 当点击工具栏删除按钮或者手动提交指令 delete 时会被触发
                        delete: ({ body }) => {
                            return fetch(`${this.serveApiUrl}/api/pub/save`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(body) }).then(response => response.json())
                        },
                        // 当点击工具栏保存按钮或者手动提交指令 save 时会被触发
                        save: ({ body }) => {
                            return fetch(`${this.serveApiUrl}/api/pub/save`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(body) }).then(response => response.json())
                        }
                    }
                },
                columns: [
                    { type: 'checkbox', title: 'ID', width: 120 },
                    { field: 'name', title: 'Name', sortable: true, titlePrefix: { message: '名称必须填写！' }, editRender: { name: 'input', attrs: { placeholder: '请输入名称' } } },
                    {
                        field: 'role',
                        title: 'Role',
                        sortable: true,
                        titlePrefix: { useHTML: true, content: '点击链接：<a class="link" href="https://vxetable.cn" target="_blank">vxe-table官网</a>' },
                        filters: [
                            { label: '前端开发', value: '前端' },
                            { label: '后端开发', value: '后端' },
                            { label: '测试', value: '测试' },
                            { label: '程序员鼓励师', value: '程序员鼓励师' }
                        ],
                        filterMultiple: false,
                        editRender: { name: 'input', attrs: { placeholder: '请输入角色' } }
                    },
                    { field: 'email', title: 'Email', width: 160, editRender: { name: '$input', props: { placeholder: '请输入邮件' } } },
                    { field: 'nickname', title: 'Nickname', editRender: { name: 'input', attrs: { placeholder: '请输入昵称' } } },
                    {
                        field: 'sex',
                        title: 'Sex',
                        filters: [
                            { label: '男', value: '1' },
                            { label: '女', value: '0' }
                        ],
                        editRender: { name: '$select', options: [], props: { placeholder: '请选择性别' } }
                    },
                    { field: 'age', title: 'Age', visible: false, sortable: true, editRender: { name: '$input', props: { type: 'number', min: 1, max: 120 } } },
                    { field: 'amount', title: 'Amount', formatter: this.formatAmount, editRender: { name: '$input', props: { type: 'float', digits: 2, placeholder: '请输入数值' } } },
                    { field: 'updateDate', title: 'Update Date', width: 160, visible: false, sortable: true, formatter: this.formatDate },
                    { field: 'createDate', title: 'Create Date', width: 160, visible: false, sortable: true, formatter: this.formatDate },
                    { title: '操作', width: 110, slots: { default: 'operate' } }
                ],
                importConfig: {
                    remote: true,
                    importMethod: this.importMethod,
                    types: ['xlsx'],
                    modes: ['insert']
                },
                exportConfig: {
                    remote: true,
                    exportMethod: this.exportMethod,
                    types: ['xlsx'],
                    modes: ['current', 'selected', 'all']
                },
                checkboxConfig: {
                    labelField: 'id',
                    reserve: true,
                    highlight: true,
                    range: true
                },
                editRules: {
                    name: [
                        { required: true, message: '名称' },
                        { min: 3, max: 50, message: '名称长度在 3 到 50 个字符' }
                    ],
                    email: [
                        { required: true, message: '邮件必须填写' }
                    ],
                    role: [
                        { required: true, message: '角色必须填写' }
                    ]
                },
                editConfig: {
                    trigger: 'click',
                    mode: 'row',
                    showStatus: true
                }
            }
        }
    },
    computed: {
        ...mapState([
            'serveApiUrl'
        ])
    },
    created() {
        this.findSexList()
    },
    methods: {
        getGoodList() {
            const page = 1
            const pageSize = 10
            ds.goodsList({ page, pageSize }).then(result => {
                console.log(result.data.data)
            })
        },
        // proxyQueryEvent() {
        //     message.success('触发查询proxyQueryEvent');
        // },
        toolbarButtonClickEvent({ code }) {
            const $grid = this.$refs.xGrid
            switch (code) {
                case 'insert_button_actived':
                    $grid.insert({
                        name: 'xxx'
                    })
                    message.success("触发新增事件")
                    break
                case 'mark_cancel_button_actived':
                    message.success("触发新增事件")
                    break
                case 'my_save_button_actived':
                    setTimeout(() => {
                        const { insertRecords, removeRecords, updateRecords } = $grid.getRecordset()
                        VXETable.modal.message({ content: `新增 ${insertRecords.length} 条，删除 ${removeRecords.length} 条，更新 ${updateRecords.length} 条`, status: 'success' })
                        this.loadData()
                    }, 100)
                    break
                case 'delete_button_actived':
                    message.success("触发删除事件")
                    break
            }
        },
        proxyDeleteEvent() {
            message.success('触发proxyDeleteEvent');
        },
        proxySaveEvent() {
            message.success('触发查询proxySaveEvent');
            this.getGoodList()
        },
        findSexList() {
            setTimeout(() => {
                const sexList = [
                    { label: '', value: '' },
                    { label: '男', value: '1' },
                    { label: '女', value: '0' }
                ]
                // 异步更新下拉选项
                this.sexList = sexList
                const $grid = this.$refs.xGrid
                if ($grid) {
                    const sexColumn = $grid.getColumnByField('sex')
                    sexColumn.editRender.options = sexList
                    const sexItem = $grid.getFormItems(4)
                    sexItem.itemRender.options = sexList
                }
            }, 100)
        },
        formatAmount({ cellValue }) {
            return cellValue ? `￥${XEUtils.commafy(XEUtils.toNumber(cellValue), { digits: 2 })}` : ''
        },
        formatDate({ cellValue }) {
            return XEUtils.toDateString(cellValue, 'yyyy-MM-dd HH:ss:mm')
        },
        checkColumnMethod({ column }) {
            if (['nickname', 'role'].includes(column.property)) {
                return false
            }
            return true
        },
        // 自定义服务端导入
        importMethod({ file }) {
            // 处理表单
            const formBody = new FormData()
            formBody.append('file', file)
            // 上传文件
            return fetch(`${this.serveApiUrl}/api/pub/import`, { method: 'POST', body: formBody }).then(response => response.join()).then(data => {
                const $grid = this.$refs.xGrid
                VXETable.modal.message({ content: `成功导入 ${data.result.insertRows} 条记录！`, status: 'success' })
                // 导入完成，刷新表格
                $grid.commitProxy('query')
            }).catch(() => {
                VXETable.modal.message({ content: '导入失败，请检查数据是否正确！', status: 'error' })
            })
        },
        // 自定义服务端导出
        exportMethod({ options }) {
            const $grid = this.$refs.xGrid
            const proxyInfo = $grid.getProxyInfo()
            // 传给服务端的参数
            const body = {
                filename: options.filename,
                sheetName: options.sheetName,
                isHeader: options.isHeader,
                original: options.original,
                mode: options.mode,
                pager: proxyInfo.pager,
                ids: options.mode === 'selected' ? options.data.map(item => item.id) : [],
                fields: options.columns.map(column => {
                    return {
                        field: column.property,
                        title: column.title
                    }
                })
            }
            // 开始服务端导出
            return fetch(`${this.serveApiUrl}/api/pub/export`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(body) }).then(response => response.json()).then(data => {
                if (data.id) {
                    VXETable.modal.message({ content: '导出成功，开始下载', status: 'success' })
                    // 读取路径，请求文件
                    fetch(`${this.serveApiUrl}/api/pub/export/download/${data.id}`)
                        .then(response => response.blob())
                        .then(blob => {
                            // 开始下载
                            VXETable.saveFile({ filename: '导出数据', type: 'xlsx', content: blob })
                        })
                }
            }).catch(() => {
                VXETable.modal.message({ content: '导出失败！', status: 'error' })
            })
        }
    }
}
</script>

<style scoped></style>
