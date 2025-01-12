<template>
  <FormulateForm>
    <div>
      <!-- Counter Arguments -->
      <forms-form-section
        :title="$t('forms.counter_arguments.question')"
        :subtitle="$t('forms.counter_arguments.help')"
      >
        <draggable
          :list="counterArguments"
          :sort="true"
          ghost-class="ghost"
          handle=".handle"
          @update="updatePriority(counterArguments, 'counter_arguments')"
        >
          <transition-group tag="ul" type="transition" name="flip-list">
            <li
              v-for="(
                counterArgument, counterArgumentIndex
              ) in counterArguments"
              :key="counterArgument.id"
              class="flex flex-col w-full justify-center items-center mb-8"
            >
              <forms-list-item-input
                :value="counterArgument.description"
                name="description"
                type="text"
                :placeholder="
                  $t('forms.counter_arguments.placeholder.description')
                "
                :validation-name="
                  $t('validation.name.counter_arguments.description')
                "
                validation="required"
                @validation="validationCounterArguments = $event"
                @focusout="
                  !validationCounterArguments.hasErrors
                    ? updateEntity(
                        'counter_arguments',
                        { description: $event },
                        counterArgument.id
                      )
                    : {}
                "
                @delete="
                  deleteEntity(
                    'counter_arguments',
                    counterArgument.id,
                    counterArguments
                  )
                "
              >
                <template #prefix>
                  <div
                    class="
                      inline-flex
                      items-center
                      space-x-2
                      pr-4
                      handle
                      cursor-move
                    "
                  >
                    <outline-menu-alt-4-icon
                      class="w-5 h-5 bg-white rounded-sm text-gray-500"
                    ></outline-menu-alt-4-icon>
                    <outline-thumb-down-icon
                      class="w-5 h-5 text-red-500"
                    ></outline-thumb-down-icon>
                    <span class="text-red-500">{{
                      $t('forms.counter_arguments.cons')
                    }}</span>
                  </div>
                </template>
              </forms-list-item-input>
              <FormulateInput
                type="group"
                remove-position="after"
                :repeatable="true"
                minimum="1"
                add-label="Konter-Argument hinzufügen"
                :value="counterArgument.negations"
              >
                <template #addmore="{ addMore }">
                  <FormulateInput
                    input-class="flex space-x-2 items-center mt-4 text-purple-500"
                    type="button"
                    @click="addMore()"
                  >
                    <outline-plus-icon class="w-6 h-6"></outline-plus-icon>
                    <span class="">{{
                      $t('forms.counter_arguments.add_counter')
                    }}</span>
                  </FormulateInput>
                </template>
                <template #default="{ index }">
                  <FormulateInput
                    :value="
                      counterArgument.negations &&
                      counterArgument.negations[index] &&
                      counterArgument.negations[index].description
                    "
                    name="description"
                    type="text"
                    :validation-name="$t('validation.name.arguments.negations')"
                    element-class="inline-flex w-full items-center"
                    input-class="border-0 w-full"
                    :placeholder="
                      $t('forms.counter_arguments.placeholder.negations')
                    "
                    validation="required"
                    @validation="validationNegations = $event"
                    @focusout="
                      !validationNegations.hasErrors
                        ? createOrUpdateNegations(
                            counterArgument.negations &&
                              counterArgument.negations[index]
                              ? counterArgument.negations[index].id
                              : null,
                            counterArgumentIndex,
                            counterArgument,
                            index,
                            $event.target.value
                          )
                        : {}
                    "
                  >
                    <template #prefix>
                      <div class="inline-flex items-center space-x-2 px-4">
                        <outline-chat-alt-2-icon
                          class="w-5 h-5"
                        ></outline-chat-alt-2-icon>
                      </div>
                    </template>
                  </FormulateInput>
                </template>
                <template #remove="{ index, removeItem }">
                  <remove-button
                    v-if="counterArgument.negations.length > 1"
                    @click="
                      removeItem()
                      deleteNegation(
                        counterArgument.negations[index].id,
                        counterArgumentIndex,
                        counterArgument.negations
                      )
                    "
                  />
                </template>
              </FormulateInput>
            </li>
          </transition-group>
        </draggable>
        <div class="border-t-2 pt-6">
          <forms-project-create-counter-arguments
            v-if="newCounterArgumentForm"
            :form-key="formKey"
            @submit="createCounterArguments($event)"
          >
          </forms-project-create-counter-arguments>
          <base-button
            v-if="!newCounterArgumentForm"
            class="
              bg-white
              text-red-500
              border border-gray-400
              flex
              items-center
              hover:text-white hover:bg-red-500
            "
            @click="newCounterArgumentForm = true"
            ><svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-4 w-4 mr-2 transform rotate-180"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M14 10h4.764a2 2 0 011.789 2.894l-3.5 7A2 2 0 0115.263 21h-4.017c-.163 0-.326-.02-.485-.06L7 20m7-10V5a2 2 0 00-2-2h-.095c-.5 0-.905.405-.905.905 0 .714-.211 1.412-.608 2.006L7 11v9m7-10h-2M7 20H5a2 2 0 01-2-2v-6a2 2 0 012-2h2.5"
              />
            </svg>
            Gegenargument hinzufügen</base-button
          >
          <div
            v-if="newCounterArgumentForm"
            class="text-red-500 text-sm cursor-pointer text-right"
            @click="newCounterArgumentForm = false"
          >
            Abbrechen
          </div>
        </div>
      </forms-form-section>

      <!-- Arguments -->
      <forms-form-section :title="$t('forms.arguments.question')">
        <draggable
          :list="argumentations"
          :sort="true"
          ghost-class="ghost"
          handle=".handle"
          @update="updatePriority(argumentations, 'arguments')"
        >
          <transition-group tag="ul" type="transition" name="flip-list">
            <li
              v-for="argumentation in argumentations"
              :key="argumentation.id"
              class="flex flex-col w-full justify-center items-center"
            >
              <forms-list-item-input
                :value="argumentation.description"
                name="description"
                type="text"
                :validation-name="$t('validation.name.arguments.description')"
                validation="required"
                @validation="validationArguments = $event"
                @focusout="
                  !validationArguments.hasErrors
                    ? updateEntity(
                        'arguments',
                        { description: $event },
                        argumentation.id
                      )
                    : {}
                "
                @delete="
                  deleteEntity('arguments', argumentation.id, argumentations)
                "
              >
                <template #prefix>
                  <div
                    class="
                      inline-flex
                      items-center
                      space-x-2
                      pr-4
                      handle
                      cursor-move
                    "
                  >
                    <outline-menu-alt-4-icon
                      class="w-5 h-5 bg-white rounded-sm text-gray-500"
                    ></outline-menu-alt-4-icon>
                    <outline-thumb-up-icon
                      class="w-5 h-5 text-green-500"
                    ></outline-thumb-up-icon>
                    <span class="text-green-500"> Pro </span>
                  </div>
                </template>
              </forms-list-item-input>
            </li>
          </transition-group>
        </draggable>

        <div class="border-t-2 pt-6">
          <forms-project-create-arguments
            v-if="newArgumentForm"
            :form-key="formKey"
            @submit="createArgumentType('arguments', argumentations, $event)"
          >
          </forms-project-create-arguments>
          <base-button
            v-if="!newArgumentForm"
            class="
              bg-white
              text-green-500
              border border-gray-400
              flex
              items-center
              hover:text-white hover:bg-green-500
            "
            @click="newArgumentForm = true"
            ><svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-4 w-4 mr-2"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M14 10h4.764a2 2 0 011.789 2.894l-3.5 7A2 2 0 0115.263 21h-4.017c-.163 0-.326-.02-.485-.06L7 20m7-10V5a2 2 0 00-2-2h-.095c-.5 0-.905.405-.905.905 0 .714-.211 1.412-.608 2.006L7 11v9m7-10h-2M7 20H5a2 2 0 01-2-2v-6a2 2 0 012-2h2.5"
              />
            </svg>
            Argument hinzufügen</base-button
          >
          <div
            v-if="newArgumentForm"
            class="text-red-500 text-sm cursor-pointer text-right"
            @click="newArgumentForm = false"
          >
            Abbrechen
          </div>
        </div>
      </forms-form-section>
    </div>
  </FormulateForm>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  onMounted,
  watch,
  useStore,
  useContext,
} from '@nuxtjs/composition-api'

import { camelCase, cloneDeep } from 'lodash'

import { IArgument, ICounterArgument, IProposal } from '~/types/apiSchema'
import editApplication from '~/composables/editApplication'
import { IValidation } from '~/types/vueFormulate'
import { RootState } from '~/store'

export default defineComponent({
  name: 'Arguments',
  setup() {
    const argumentations = ref<IArgument[]>([])
    const counterArguments = ref<ICounterArgument[]>([])

    /*
 workaround for resetting form and validation because
 $formulate plugin is not support for vue 3 composition aoi now
 */
    const formKey = ref(1)

    const store = useStore<RootState>()
    const context = useContext()

    const {
      createProjectEntity,
      deleteProjectEntity,
      updateProjectEntity,
      project,
    } = editApplication()

    onMounted(() => {
      if (project.value?.arguments) {
        argumentations.value = cloneDeep(project.value.arguments)
        // @ts-ignore
        argumentations.value.sort((a, b) => b.priority - a.priority)
      }
      if (project.value?.counterArguments) {
        counterArguments.value = cloneDeep(project.value.counterArguments)
        // @ts-ignore
        counterArguments.value.sort((a, b) => b?.priority - a.priority)
      }
    })

    watch(
      project,
      (currentValue) => {
        argumentations.value = cloneDeep(currentValue?.arguments || [])
        // @ts-ignore
        argumentations.value.sort((a, b) => b.priority - a.priority)
        counterArguments.value = cloneDeep(currentValue?.counterArguments || [])
        // @ts-ignore
        counterArguments.value.sort((a, b) => b.priority - a.priority)
      },
      { deep: true }
    )

    const validationArguments = ref<IValidation>({ hasErrors: false })
    const validationCounterArguments = ref<IValidation>({ hasErrors: false })
    const validationNegations = ref<IValidation>({ hasErrors: false })

    const createCounterArguments = async (
      formData: IArgument | ICounterArgument
    ) => {
      await createArgumentType(
        'counter_arguments',
        counterArguments.value,
        formData
      )
      newCounterArgumentForm.value = false
    }

    const createArgumentType = async (
      endpoint: string,
      projectProperty: (IArgument | ICounterArgument)[],
      formData: IArgument | ICounterArgument
    ) => {
      if (project.value) {
        const payload = {
          ...formData.value,
          project: project.value['@id'],
        }
        return await createProjectEntity<IArgument | ICounterArgument>(
          endpoint,
          projectProperty,
          payload
        ).then((res) => {
          formKey.value++
          newArgumentForm.value = false
          return res
        })
      }
    }

    const updateEntity = async (
      endpoint: string,
      data: IProposal | IArgument | ICounterArgument,
      id: string | number
    ) => {
      if (project.value && typeof project.value['@id'] === 'string') {
        const payload: IProposal | IArgument | ICounterArgument = {
          ...data,
          project: project.value['@id'],
        }
        await updateProjectEntity<IArgument | IProposal | ICounterArgument>(
          endpoint,
          id,
          payload
        )
      }
    }

    const deleteEntity = async (
      endpoint: string,
      id: number | string,
      projectProperty: IArgument | ICounterArgument
    ) => {
      // @ts-ignore
      await deleteProjectEntity<IArgument | ICounterArgument>(
        endpoint,
        id,
        projectProperty
      )
    }

    const updatePriority = async (
      entity: ICounterArgument | IArgument,
      endpoint: string
    ) => {
      const allAsyncResults: Promise<any>[] = []

      for (let index = 0; index < entity.length; index++) {
        const payload: ICounterArgument | IArgument = {
          priority: entity.length - (index + 1),
        }
        // @ts-ignore
        const asyncResult: any = await context.$api[endpoint].update(
          entity[index].id,
          payload
        )

        allAsyncResults.push(asyncResult)
      }
      await Promise.all(allAsyncResults).then((res) => {
        store.commit('projects/SET_PROJECT_PROPERTY', [
          camelCase(endpoint),
          res.map((e) => e.data),
        ])
      })
    }

    // @ts-ignore
    const deleteNegation = async (id, counterArgumentIndex, negations) => {
      // @ts-ignore
      await context.$api.negations.delete(id).then(() => {
        // @ts-ignore
        const data = negations.filter((e) => e.id !== id)
        store.commit('projects/SET_PROJECT_PROPERTY', [
          `counterArguments[${counterArgumentIndex}].negations`,
          data,
        ])
      })
    }

    const createOrUpdateNegations = async (
      id: string,
      counterArgumentIndex: number,
      counterArgument: ICounterArgument,
      negationIndex: number,
      value: string
    ) => {
      const payload = {
        description: value,
        counterArgument: counterArgument['@id'],
      }
      if (id) {
        // @ts-ignore
        await context.$api.negations.update(id, payload).then((res) => {
          store.commit('projects/SET_PROJECT_PROPERTY', [
            `counterArguments[${counterArgumentIndex}].negations[${negationIndex}]`,
            res.data,
          ])
        })
      } else {
        // @ts-ignore
        await context.$api.negations.create(payload).then((res) => {
          counterArguments.value[counterArgumentIndex].negations.push(res.data)
          const data = counterArguments.value[counterArgumentIndex].negations
          store.commit('projects/SET_PROJECT_PROPERTY', [
            `counterArguments[${counterArgumentIndex}].negations`,
            data,
          ])
        })
      }
    }

    const newArgumentForm = ref(false)
    const newCounterArgumentForm = ref(false)

    return {
      argumentations,
      counterArguments,
      formKey,
      validationArguments,
      validationCounterArguments,
      validationNegations,
      createArgumentType,
      updateEntity,
      deleteEntity,
      deleteNegation,
      updatePriority,
      createCounterArguments,
      createOrUpdateNegations,
      newArgumentForm,
      newCounterArgumentForm,
    }
  },
})
</script>
