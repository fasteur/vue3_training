<template>
    <div class="container__wrapper">
        <el-form
            :model="userForm"
            :rules="rules"
            label-position="left"
            label-width="120px"
            class="container__inner"
        >
            <el-form-item :label="labels.name" prop="name">
                <el-input
                    size="medium"
                    type="text"
                    v-model="userForm.name"
                    placeholder="Ex: Asteur"
                    clearable
                />
            </el-form-item>

            <el-form-item :label="labels.firstName" prop="firstName">
                <el-input
                    type="text"
                    v-model="userForm.firstName"
                    placeholder="Ex: Florian"
                    clearable
                />
            </el-form-item>

            <el-form-item :label="labels.age" prop="age">
                <el-input-number
                    v-model="userForm.age"
                    :min="0"
                    :max="120"
                    label="age"
                    controls-position="right"
                >
                </el-input-number>
            </el-form-item>

            <el-form-item prop="email" :label="labels.email">
                <el-input v-model="userForm.email"></el-input>
            </el-form-item>

            <el-form-item :label="labels.password" prop="password">
                <el-input
                    type="text"
                    :show-password="true"
                    v-model="userForm.password"
                    placeholder="Ex: 65fddfyyhb$"
                    clearable
                />
            </el-form-item>
        </el-form>
    </div>
</template>

<script lang="ts">
import { ref, toRefs, watch, onMounted, computed, defineComponent, PropType, inject } from 'vue'
import { validators } from '@/utils/form/validator-rules'
import { FormGroup } from '@/utils/form/form-group'
import { Path, TranslateResult } from 'vue-i18n';

declare type RegisterFormPropeties = { 
    name: string,
    firstName: string,
    password: string,
    email: string,
    age: number
}

export default defineComponent({
    props: {
        form: {
            type: Object as PropType<RegisterFormPropeties>,
            default: null,
        }
    },
    emits: ['update:form'],
    setup(props, { emit }) {
        // Injects
        const translate =  inject<(key: Path) => TranslateResult>('i18nTranslate')

        // Props
        const { form } = toRefs(props)

        // Datas
        const formLabelWidth = '20'

        const formGroup = new FormGroup({ 
            name: {
                value: '',
                rules: [validators.required()],
            },
            firstName: {
                value: '',
                rules: [validators.required()],
            },
            password: {
                value: '',
                rules: [validators.required()],
            },
            email: {
                value: '',
                rules: [validators.required(), validators.email()],
            },
            age: {
                value: 0,
                rules: [validators.required()],
            },
        });

        const userForm = ref(formGroup.value);
        const rules = ref(formGroup.rules);

        // LifeCycle Hooks
        onMounted(() => {
            userForm.value = form.value
        })

        // Watchers
        watch(form, function (val: RegisterFormPropeties) {
            userForm.value = val
        })

        watch(userForm, function (val) {
            emit('update:form', val)
        })

        // Computed Properties
        const labels = computed(() => {
            return {
                name: translate!('REGISTER_FORM.COLUMNS.NAME'),
                firstName: translate!('REGISTER_FORM.COLUMNS.FIRST_NAME'),
                password: translate!('REGISTER_FORM.COLUMNS.PASSWORD'),
                email: translate!('REGISTER_FORM.COLUMNS.EMAIL'),
                age: translate!('REGISTER_FORM.COLUMNS.AGE'),
            }
        })

        return { 
            userForm,
            formLabelWidth,
            rules,
            labels,
        }
    },
})
</script>

<style lang="scss">
@import '@/assets/style.scss';

.el-form {
    width: 460px;
}
</style>