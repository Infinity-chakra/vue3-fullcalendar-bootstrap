<script setup lang="ts">
import { storeToRefs } from 'pinia';
import { addHours, format } from "date-fns";
import { useCalendarStore } from '@/stores/calendar';
import { computed, ref, watch } from 'vue';

const calendarStore = useCalendarStore()
const { modal, selectedSlot } = storeToRefs(calendarStore)
const { setModal, setAddedEvents, setSelectedSlot } = calendarStore

const eventTitle = ref('')
const startDateTime = ref<string | Date | null>( new Date() )
const endDateTime = ref<string | Date | null>( addHours(new Date(), 1) )
const contentText = ref('')
const titleState = computed(() => (eventTitle.value?.length > 2 ? true : false))

const addEvent = () => {
  setAddedEvents({
    title: eventTitle.value,
    start: startDateTime.value,
    end: endDateTime.value,
    contentText: contentText.value
  }, 'add')
  
  closeModal()
}

const closeModal = () => {
  modal.value && setModal()
  selectedSlot.value.modal && setSelectedSlot({
    modal: false,
    times: {
      start: null,
      end: null
    }
  })
  eventTitle.value = ''
}

watch(() => selectedSlot.value.modal, () => {
  if(selectedSlot.value.modal) {
    startDateTime.value = selectedSlot.value.times.start
    endDateTime.value = selectedSlot.value.times.end
  }
  console.log('close--->', startDateTime.value, selectedSlot.value.times.start)
})
</script>

<template>
  <teleport to="body">
    <div class="modal-wrapper" v-if="modal || selectedSlot.modal">
      <BCard 
        header="Calendar Modal" 
        header-tag="header" 
        class="fc-modal text-center bg-white fs-5 p-0 min"
        title="Add Event"
        style="min-width: 400px;"
      >
        <BCardBody class="fs-6 pt-0">
          <div role="group" class="text-start">
            <BCol cols="12" class="p-1">
              <label for="title">Title:</label>
              <BFormInput id="title" v-model="eventTitle" placeholder="Enter event title" :state="titleState" trim />
              <BFormInvalidFeedback id="input-live-feedback"> Enter at least 3 letters </BFormInvalidFeedback>
            </BCol>
            <BCol cols="12" class="p-1">
              <label for="start">Start:</label>
              <VueDatePicker aria-label="start" v-model="startDateTime" placeholder="Pick the start time" />
            </BCol>
            <BCol cols="12" class="p-1">
              <label for="end">Until:</label>
              <VueDatePicker aria-label="end" v-model="endDateTime" placeholder="Pick the End time" />
            </BCol>
            <BCol cols="12" class="p-1">
              <BFormTextarea
                id="textarea"
                v-model="contentText"
                placeholder="Enter something..."
                rows="3"
                max-rows="6"
              />
            </BCol>
          </div>

        </BCardBody>

        <template v-slot:footer>
          <div tag="footer" class="d-flex justify-content-end">
            <BButtonGroup>
              <BButton variant="primary" @click="closeModal()">Close</BButton>
              <BButton variant="success" @click="addEvent()">Add</BButton>
            </BButtonGroup>
          </div>
        </template>
      </BCard>
    </div>

  </teleport>
</template>

<style scoped lang="scss">
.modal-wrapper {
  width: 100%;
  height: 100vh;
  background-color: #0000006a;
  position: fixed;
  z-index: 10;
  top: 0;
  left: 0;
}

.fc-modal {
  position: fixed;
  max-width: 600px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
