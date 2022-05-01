<template>
  <div>
    <div class="container">
      <a href="#" class="btn btn-primary" @click.prevent="tampilFormEvent">
        Buat Event Baru
      </a>
      <hr>

      <b-modal id="bv-modal-event" hide-footer>
        <template #modal-title>
          {{title}}
        </template>
        <form @submit.prevent="simpanEvent">
          <div class="form-group">
            <label for="" class="control-label">Title</label>
            <input type="text" v-model="form.title" class="form-control">
          </div>
          <div class="form-group">
            <label for="" class="control-label">Start Date</label>
            <input type="date" v-model="form.start_date" class="form-control">
          </div>
          <div class="form-group">
            <label for="" class="control-label">End Date</label>
            <input type="date" v-model="form.end_date" class="form-control">
          </div>
          <div class="form-group">
            <label for="" class="control-label">Start Time</label>
            <input type="time" v-model="form.start_time" class="form-control">
          </div>
          <div class="form-group">
            <label for="" class="control-label">End Time</label>
            <input type="time" v-model="form.end_time" class="form-control">
          </div>
          <div class="form-group">
            <button class="btn btn-primary btn btn-block" type="submit">Simpan</button>
          </div>
        </form>
        
      </b-modal>

      <FullCalendar :options="calendarOptions"  ref="calendar"  @rerender-events="generateEvent" @event-selected="eventSelected"/>


    </div>
  </div>
</template>

<script>
import '@fullcalendar/core/vdom'
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import timeGridPlugin from '@fullcalendar/timegrid'
import moment from 'moment'

export default {
  components: {
    FullCalendar,
  },
  data(){
    return {
      api_url:'http://127.0.0.1:8000',
      calendarOptions: {
        plugins: [ dayGridPlugin, interactionPlugin, timeGridPlugin ],
        initialView: 'timeGridWeek',
        validRange: {
            start: new Date()
        },
        slotLabelFormat: [
            {
                hour: '2-digit',
                minute: '2-digit',
                hour12:false
            }
        ],
        locale: 'en',
        dateClick: this.dayClick,
        selectable:true,
        eventClick: this.eventSelected,
        events: [],
      },
      title:'',
      form:{
        type:'add',
        id:'',
        title:'',
        start_date:'',
        end_date:'',
        start_time:'',
        end_time:''
      }
    }
  },
  filters: {
    moment: function (date) {
      return moment(date).format('YYYY-MM-DD');
    }
  },
  mounted(){
    this.generateEvent()
  },
  methods:{
    generateEvent(){
      this.calendarOptions.events = []

      this.$axios.get(this.api_url+'/api/event')
        .then(resp => {
          for(var a=0; a<resp.data.length; a++)
          {
            this.calendarOptions.events.push({
              id: resp.data[a].id,
              title: resp.data[a].title,
              start: resp.data[a].start_date+'T'+resp.data[a].start_time.replace(/\s+/g, ' ').trim(),
              end: resp.data[a].end_date+'T'+resp.data[a].end_time.replace(/\s+/g, ' ').trim(),
              allDay:false,
            })
          }
        })
    },
    tampilFormEvent(){
      this.title = "Buat Event Baru"
      
      this.form.form = {
        type:'add',
        id:'',
        title:'',
        start_date:'',
        end_date:'',
        start_time:'',
        end_time:''
      }

      this.$bvModal.show('bv-modal-event')
    },
    
    simpanEvent(){
      this.$axios.post(this.api_url+'/api/store-event', this.form)
        .then(resp => {
          this.$bvModal.hide('bv-modal-event')
          this.generateEvent()
        })
    },

    eventSelected(e){
      this.title = "Edit Event"

      this.form = {
        id: e.event.id,
        type:'edit',
        title:e.event.title,
        start_date:moment(e.event.start).format('YYYY-MM-DD'),
        end_date:moment(e.event.end).format('YYYY-MM-DD'),
        start_time:moment(e.event.start).format('hh:mm:ss'),
        end_time:moment(e.event.end).format('hh:mm:ss')
      }

      this.$bvModal.show('bv-modal-event')
    }

  }
}
</script>