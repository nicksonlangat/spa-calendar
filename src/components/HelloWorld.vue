<template>
  <v-row class="fill-height">
    <v-col>
      <v-sheet height="64">
        <v-toolbar
          flat
        >
         <v-btn
            outlined
            class="mr-4"
            color="grey darken-2"
            @click="dialog=true"
          >
            New Event
          </v-btn>
          <v-btn
            outlined
            class="mr-4"
            color="grey darken-2"
            @click="setToday"
          >
            Today
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="prev"
          >
            <v-icon small>
              mdi-chevron-left
            </v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="next"
          >
            <v-icon small>
              mdi-chevron-right
            </v-icon>
          </v-btn>
          <v-toolbar-title v-if="$refs.calendar">
            {{ $refs.calendar.title }}
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-menu
            bottom
            right
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                outlined
                color="grey darken-2"
                v-bind="attrs"
                v-on="on"
              >
                <span>{{ typeToLabel[type] }}</span>
                <v-icon right>
                  mdi-menu-down
                </v-icon>
              </v-btn>
            </template>
            <v-list>
              <v-list-item @click="type = 'day'">
                <v-list-item-title>Day</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'week'">
                <v-list-item-title>Week</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'month'">
                <v-list-item-title>Month</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = '4day'">
                <v-list-item-title>4 days</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-toolbar>
      </v-sheet>
      <v-dialog v-model="dialog" max-width="500">
        <v-card>
          <v-container>
            <v-form @submit.prevent="addEvent">
              <v-text-field v-model="name" type="text" label="EVENT NAME"></v-text-field>
              <v-text-field v-model="details" type="text" label="DESCRIPTION"></v-text-field>
              <v-text-field v-model="start" type="date" label="START DATE"></v-text-field>
              <v-text-field v-model="end" type="date" label="END DATE"></v-text-field>
              <v-text-field v-model="color" type="color" label="COLOR (click to open color menu)"></v-text-field>
              <v-btn type="submit" color="primary" class="mr-4" @click.stop="dialog = false">
                create event
              </v-btn>
            </v-form>
          </v-container>
        </v-card>
      </v-dialog>

      <v-dialog v-model="dialogDate" max-width="500">
        <v-card>
          <v-container>
            <v-form @submit.prevent="addEvent">
              <v-text-field v-model="name" type="text" label="EVENT NAME"></v-text-field>
              <v-text-field v-model="details" type="text" label="DESCRIPTION"></v-text-field>
              <v-text-field v-model="start" type="date" label="START DATE"></v-text-field>
              <v-text-field v-model="end" type="date" label="END DATE"></v-text-field>
              <v-text-field v-model="color" type="color" label="COLOR (click to open color menu)"></v-text-field>
              <v-btn type="submit" color="primary" class="mr-4" @click.stop="dialog = false">
                create event
              </v-btn>
            </v-form>
          </v-container>
        </v-card>
      </v-dialog>
      
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="focus"
          color="primary"
          :events="events"
          :event-color="getEventColor"
          :type="type"
          @click:event="showEvent"
          @click:more="viewDay"
          @click:date="viewDay"
          @change="getEvents"
        ></v-calendar>
        <v-menu
          v-model="selectedOpen"
          :close-on-content-click="false"
          :activator="selectedElement"
          offset-x
        >
          <v-card
            color="grey lighten-4"
            min-width="350px"
            flat
          >
            <v-toolbar
              :color="selectedEvent.color"
              dark
            >
              <v-btn @click="deleteEvent(selectedEvent.id)" icon>
                <v-icon>mdi-delete</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
              <!-- <v-btn icon>
                <v-icon>mdi-heart</v-icon>
              </v-btn> -->
              <!-- <v-btn icon>
                <v-icon>mdi-dots-vertical</v-icon>
              </v-btn> -->
            </v-toolbar>
            <v-card-text>
             <form v-if="currentlyEditing!==selectedEvent.id">
                <span v-html="selectedEvent.details"></span>
             </form>
             <form v-else>
               <textarea-autosize v-model="selectedEvent.details"
               type='text'
               style="width:100%"
               :min-height='100'
               placeholder='Add note'
               >

               </textarea-autosize>
             </form>
            </v-card-text>
            <v-card-actions>
              <v-btn
                text
                color="secondary"
                @click="selectedOpen = false"
              >
                Close
              </v-btn>
              <v-btn
                text
                color="secondary"
               v-if="currentlyEditing!==selectedEvent.id"
               @click.prevent="editEvent(selectedEvent)"
              >
                Edit
              </v-btn>
              <v-btn
                text
                color="secondary"
               v-else
               @click.prevent="updateEvent(selectedEvent)"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-sheet>
    </v-col>
  </v-row>
</template>
<script>
import axios from 'axios'
import moment from 'moment';
  export default {
    data: () => ({
      focus: '',
      type: 'week',
      typeToLabel: {
        month: 'Month',
        week: 'Week',
        day: 'Day',
        '4day': '4 Days',
      },
      selectedEvent: {},
      selectedElement: null,
       currentlyEditing: null,
      selectedOpen: false,
      events: [],
      colors: ['blue', 'indigo', 'deep-purple', 
      'cyan', 'green', 'orange', 'grey darken-1'],
      names: ['Meeting', 'Holiday', 'PTO', 'Travel', 
      'Event', 'Birthday', 'Conference', 'Party'],
      name:null,
      details:null,
      start:null,
      end:null,
      color:'#1976D2',
       dialog: false,
    dialogDate: false
    }),
    mounted () {
      this.getEvents()
      // this.$refs.calendar.checkChange()
    },
    methods: {

      addEvent(){
        const event ={
          name:this.name,
          details:this.details,
          start:this.start,
          end:this.end,
          color:this.color,
        }
        console.log(event)
        axios.post('https://techwithnick.com/events/api/', event).then(res=>{
          // console.log(res)
          this.getEvents()
        })
      },
      getEvents(){ 
      axios.get('https://techwithnick.com/events/api').then(res=>{
          // console.log(res.data.results)
          // const events=[]
          // res.data.forEach(item=>{
          //   console.log(moment(item.start)
          //   .format('YYYY-MM-DD H:mm'))
          //   let appdata=item
          //   appdata.start=moment(item.start)
          //   .format('YYYY-MM-DD H:mm')
          //    appdata.end=moment(item.end)
          //   .format('YYYY-MM-DD H:mm')
          //   events.push(appdata)
          // })
          // this.events=events
          this.events=res.data.results
          // moment(this.end).format('MM-DD-YYYY');
        })
      },
      editEvent(env){
        this.currentlyEditing=env.id
      },
      deleteEvent(id){
        axios.delete(`https://techwithnick.com/events/api/${id}`).then(res=>{
          console.log(res.status)
          this.selectedOpen=false
          this.getEvents()
        })
      },
      updateEvent(env){
        const event={
          details:env.details
        }
        console.log(event)
        axios.patch(`https://techwithnick.com/events/api/${this.currentlyEditing}/`, event).then(res=>{
          console.log(res.data)
          this.selectedOpen=false
          this.currentlyEditing=null
        })
      },

      viewDay ({ date }) {
        this.focus = date
        this.type = 'day'
      },
      getEventColor (event) {
        return event.color
      },
      setToday () {
        this.focus = ''
      },
      prev () {
        this.$refs.calendar.prev()
      },
      next () {
        this.$refs.calendar.next()
      },
      showEvent ({ nativeEvent, event }) {
        const open = () => {
          this.selectedEvent = event
          this.selectedElement = nativeEvent.target
          requestAnimationFrame(() => requestAnimationFrame(() => this.selectedOpen = true))
        }

        if (this.selectedOpen) {
          this.selectedOpen = false
          requestAnimationFrame(() => requestAnimationFrame(() => open()))
        } else {
          open()
        }

        nativeEvent.stopPropagation()
      },
      updateRange ({ start, end }) {
        const events = []

        const min = new Date(`${start.date}T00:00:00`)
        const max = new Date(`${end.date}T23:59:59`)
        const days = (max.getTime() - min.getTime()) / 86400000
        const eventCount = this.rnd(days, days + 20)

        for (let i = 0; i < eventCount; i++) {
          const allDay = this.rnd(0, 3) === 0
          const firstTimestamp = this.rnd(min.getTime(), max.getTime())
          const first = new Date(firstTimestamp - (firstTimestamp % 900000))
          const secondTimestamp = this.rnd(2, allDay ? 288 : 8) * 900000
          const second = new Date(first.getTime() + secondTimestamp)

          events.push({
            name: this.names[this.rnd(0, this.names.length - 1)],
            start: first,
            end: second,
            color: this.colors[this.rnd(0, this.colors.length - 1)],
            timed: !allDay,
          })
        }

        this.events = events
      },
      rnd (a, b) {
        return Math.floor((b - a + 1) * Math.random()) + a
      },
    },
  }
</script>