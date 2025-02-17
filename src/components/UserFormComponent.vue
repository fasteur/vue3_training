<template>
    <div>
        <el-form>
            <el-form-item
                :label="labels.name"
                :label-width="state.formLabelWidth"
                required
            >
                <el-input
                    type="text"
                    v-model="state.userForm.name"
                    placeholder="Ex: Asteur"
                    @blur="v$.name.$touch"
                    clearable
                />
                <template v-if="v$.name.$errors">
                    <div
                        v-for="error in v$.name.$errors"
                        :key="error.$uid"
                        class="el-form-item__error pl-1"
                    >
                        {{ error?.$message }}
                    </div>
                </template>
            </el-form-item>

            <el-form-item
                :label="labels.firstName"
                :label-width="state.formLabelWidth"
                required
            >
                <el-input
                    type="text"
                    v-model="state.userForm.firstName"
                    placeholder="Ex: Florian"
                    @blur="v$.firstName.$touch"
                    clearable
                />
                <template v-if="v$.firstName.$errors">
                    <div
                        v-for="error in v$.firstName.$errors"
                        :key="error.$uid"
                        class="el-form-item__error pl-1"
                    >
                        {{ error?.$message }}
                    </div>
                </template>
            </el-form-item>

            <el-form-item
                :label="labels.age"
                :label-width="state.formLabelWidth"
                required
            >
                <el-input-number
                    v-model.number="state.userForm.age"
                    :min="0"
                    :max="120"
                    label="age"
                    @blur="v$.age.$touch"
                    controls-position="right"
                >
                </el-input-number>
                <template v-if="v$.age.$errors">
                    <div
                        v-for="error in v$.age.$errors"
                        :key="error.$uid"
                        class="el-form-item__error pl-1"
                    >
                        {{ error?.$message }}
                    </div>
                </template>
            </el-form-item>
        </el-form>
        <div class="r-flex-end mt-4 mr-2">
            <el-button
                :disabled="v$.$invalid"
                native-type="submit"
                type="primary"
                @click="submitForm()"
            >
                {{ $t('GENERAL.SUBMIT') }}
            </el-button>
            <el-button
                :disabled="v$.$invalid"
                native-type="button"
                type="primary"
                plain
                @click="clearForm()"
            >
                {{ $t('GENERAL.RESET') }}
            </el-button>
        </div>
    </div>
</template>

<script lang="ts">
import {
    toRefs,
    watch,
    onMounted,
    reactive,
    computed,
    inject,
    PropType,
    onUnmounted,
} from 'vue'
import { Path, TranslateResult } from 'vue-i18n'
import { ValiduetorMessages } from '@/components/login/LoginFormComponent.vue'
import useVuelidate from '@vuelidate/core'
import { IKeyValue } from '../models/interfaces/key-value.interface'
import { helpers, required } from '@vuelidate/validators'
import { User } from '../models'

export interface UserForm {
    name: string
    firstName: string
    age: number
}

interface UserFormComponentDataState {
    userForm: UserForm
    formLabelWidth: string
}

interface UserFormComponentProps {
    user: User
}

interface UserFormComponentEmits {
    emit: (event: 'submitForm', ...args: any[]) => void
}

export default {
    props: {
        user: {
            type: Object as PropType<User>,
            default: null,
        }
    },
    emits: ['submitForm'],
    setup(
        props: Readonly<UserFormComponentProps>,
        { emit }: UserFormComponentEmits
    ) {
        // Injects
        const translate = inject<(key: Path) => TranslateResult>(
            'i18nTranslate'
        )

        // Props
        const { user } = toRefs(props)

        // State
        const state: UserFormComponentDataState = reactive({
            userForm: {
                name: '',
                firstName: '',
                age: 0,
            },
            formLabelWidth: '120px',
        })

        // Datas
        const validatorMessages: ValiduetorMessages = reactive({
            required: helpers.withMessage(
                () => labels.value.validators.required,
                required
            ),
        })

        // Computed Properties
        const labels = computed(() => {
            return {
                name: translate!('USER_LIST.COLUMNS.NAME'),
                firstName: translate!('USER_LIST.COLUMNS.FIRST_NAME'),
                age: translate!('USER_LIST.COLUMNS.AGE'),
                validators: {
                    required: translate!('VALIDATOR.REQUIRED'),
                },
            }
        })

        const rules = computed(() => {
            return {
                name: {
                    required: validatorMessages.required,
                },
                firstName: {
                    required: validatorMessages.required,
                },
                age: {
                    required: validatorMessages.required,
                },
            }
        })

        // Datas
        const v$ = useVuelidate(
            rules.value as IKeyValue,
            state.userForm as IKeyValue
        )

        // LifeCycle Hooks
        onMounted(() => {
            if (user && user.value && user.value.id) {
                const { name, firstName, age } = user.value
                state.userForm.name = name
                state.userForm.firstName = firstName
                state.userForm.age = age
            }
        })

        // Watchers
        watch(user, function (val: User) {
            if (val && val.id) {
                const { name, firstName, age } = user.value
                state.userForm.name = name
                state.userForm.firstName = firstName
                state.userForm.age = age
            }
            state.userForm = val
        })

        // Methods
        const submitForm = () => {
            if (!v$.value.$invalid) {
                emit('submitForm', new User({ 
                    id: user.value.id ? user.value.id : undefined, 
                    name: state.userForm.name,
                    firstName: state.userForm.firstName,
                    age: state.userForm.age,
                }))
            }
        }

        const clearForm = () => {
            state.userForm.name = ''
            state.userForm.firstName = ''
            state.userForm.age = 0
            v$.value.$reset()
        }

        return {
            state,
            labels,
            v$,
            submitForm,
            clearForm,
        }
    },
}
</script>
<style lang="scss">
@import '@/assets/style.scss';

.modal__footer {
    text-align: right;
}
</style>