<template>
    <AuthenticatedLayout>
        <div class="text-white pl-0 p-2">
            <div class="flex justify-between items-center">
                <h4 class="text-xl font-bold">
                    Casts
                    <small class="ml-2 text-gray-500 font-thin text-[13px]"
                        >{{ casts.total }} Total</small
                    >
                </h4>
                <!-- Bradcrumb -->
                <Breadcrumb>
                    <el-breadcrumb-item
                        ><a href="/">Movie Lists</a></el-breadcrumb-item
                    >
                    <el-breadcrumb-item>Casts</el-breadcrumb-item>
                </Breadcrumb>
            </div>

            <!-- Form -->
            <div class="flex items-center justify-between my-4">
                <div class="flex items-center">
                    <el-button @click="addNew" type="primary">
                        <font-awesome-icon icon="fa-solid fa-plus" />
                        <span class="ml-1">Add New</span>
                    </el-button>
                    <div class="flex gap-2 ml-4">
                        <el-input
                            v-model="form.castId"
                            placeholder="Movie DB Cast ID"
                        />
                        <el-button
                            type="success"
                            :disabled="form.castId === '' || form.processing"
                            @click="generateItem"
                        >
                            <font-awesome-icon :icon="['fas', 'server']" />
                            <span class="ml-1">Generate</span>
                        </el-button>
                    </div>
                </div>
                <div>
                    <el-input
                        placeholder="Search Casts..."
                        v-model="param.search"
                    />
                </div>
            </div>

            <!-- Main -->

            <div class="relative overflow-x-auto">
                <el-table
                    :data="casts.data"
                    :default-sort="{ prop: 'name', order: 'ascending' }"
                    table-layout="fixed"
                >
                    <el-table-column type="index" label="Sr." width="100" />
                    <el-table-column label="Profile" width="100">
                        <template #default="scope">
                            <el-image
                                class="h-8 w-8 rounded-full border border-secondary-600 shadow-md"
                                :src="scope.row.profile"
                                fit="cover"
                            />
                        </template>
                    </el-table-column>
                    <el-table-column prop="name" label="Name" sortable />

                    <el-table-column prop="gender" label="Gender" />
                    <el-table-column prop="birthday" label="Birthday" />

                    <el-table-column
                        prop="created_at"
                        label="Created At"
                        sortable
                        align="center"
                    />
                    <el-table-column label="Actions" align="center">
                        <template #default="scope">
                            <el-tooltip
                                class="box-item"
                                content="Edit"
                                placement="top"
                            >
                                <el-button
                                    type="warning"
                                    style="margin-bottom: 5px"
                                    circle
                                    @click="handleEdit(scope.row)"
                                >
                                    <font-awesome-icon
                                        icon="fa-regular fa-pen-to-square"
                                    />
                                </el-button>
                            </el-tooltip>
                            <el-tooltip
                                class="box-item"
                                content="Delete"
                                placement="top"
                            >
                                <el-button
                                    type="danger"
                                    circle
                                    style="margin-bottom: 5px"
                                    @click="deleteHandler(scope.row.id)"
                                >
                                    <font-awesome-icon
                                        :icon="['fas', 'trash-can']"
                                    />
                                </el-button>
                            </el-tooltip>
                        </template>
                    </el-table-column>
                </el-table>

                <div class="my-5 flex items-center justify-center">
                    <el-pagination
                        :hide-on-single-page="casts.total < param.page_size"
                        @size-change="onSizeChange"
                        @current-change="onCurrentChange"
                        :page-size="param.page_size"
                        :background="true"
                        :page-sizes="pageList"
                        :current-page="param.page"
                        :layout="`total,sizes,prev,pager,next,jumper`"
                        :total="casts.total"
                    />
                </div>
            </div>
        </div>

        <Dialog
            :show="showDialog"
            @closed="closeDialog"
            :title="dialog.dialogTitle"
            :data="dialog.dialogData"
        />
    </AuthenticatedLayout>
</template>

<script>
import Breadcrumb from "@/Components/Breadcrumb.vue";
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import { reactive, toRefs, watch } from "vue";
import Dialog from "./Dialog.vue";
import { router, useForm } from "@inertiajs/vue3";
import { ElMessage, ElMessageBox } from "element-plus";
import debounce from "lodash.debounce";
export default {
    props: ["casts"],
    components: {
        Breadcrumb,
        AuthenticatedLayout,
        Dialog,
    },
    setup(props) {
        const state = reactive({
            showDialog: false,
            isLoading: false,
            dialog: {
                dialogTitle: "",
                dialogData: {},
            },
            pageList: [10, 20, 60, 80, 100],
            param: {
                page: 1,
                page_size: 10,
                search: "",
            },
        });

        const form = useForm({
            castId: "",
        });

        const addNew = () => {
            state.dialog.dialogTitle = "Create";
            state.dialog.dialogData = {};
            state.showDialog = true;
        };

        const handleEdit = (row) => {
            state.dialog.dialogTitle = "Edit";
            state.dialog.dialogData = JSON.parse(JSON.stringify(row));
            state.showDialog = true;
        };

        const generateItem = () => {
            form.post(route("admin.casts.generate"), {
                onSuccess: (page) => {
                    form.reset();
                    ElMessage.success(page.props.flash.success);
                },
            });
        };

        watch(
            () => state.param.search,
            debounce(() => {
                getData();
            }, 500)
        );

        const onSizeChange = (val) => {
            state.param.page_size = val;
            getData();
        };

        const onCurrentChange = (val) => {
            state.param.page = val;
            getData();
        };

        function getData() {
            router.get("/admin/casts", state.param, {
                preserveScroll: true,
                preserveState: true,
                replace: true,
            });
        }

        const deleteHandler = (id) => {
            ElMessageBox.confirm(
                "Are you sure you want to delete?",
                "Warning",
                {
                    confirmButtonText: "Confirm",
                    cancelButtonText: "Cancel",
                    type: "warning",
                    draggable: true,
                    closeOnClickModal: false,
                }
            )
                .then(() => {
                    form.delete(route("admin.casts.destroy", id), {
                        onSuccess: (page) => {
                            ElMessage.success(page.props.flash.success);
                        },
                    });
                })
                .catch(() => {
                    ElMessage({
                        type: "info",
                        message: "Cancel",
                    });
                });
        };

        const closeDialog = () => {
            state.showDialog = false;
        };

        return {
            ...toRefs(state),
            addNew,
            closeDialog,
            handleEdit,
            deleteHandler,
            generateItem,
            form,
            onSizeChange,
            onCurrentChange,
        };
    },
};
</script>
