<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {username: String},
  data() {
    return {
      meetings: [],
      participants: []
    };
  },
  methods: {
    addNewMeeting(meeting) {
      this.meetings.push({id: meeting.id, title: meeting.title, description: meeting.description, participants: []})

      axios
          .post('/api/meetings', meeting)
          .then(response => {console.log(response.data.id);console.log(response.data);})
          .catch(response => {
            this.message = "Nie udało się dodać spotkania, spróbuj ponownie";
          });
    },

    addMeetingParticipant(meeting) {
      meeting.participants.push(this.username);
      let url = 'api/meetings/' + meeting.id + '/participants/';

      axios
          .get('api/participants/' + this.username)
          .then(response => axios.post(url, response.data)
      );
    },

    removeMeetingParticipant(meeting) {
      meeting.participants.splice(meeting.participants.indexOf(this.username), 1);

      let url = 'api/meetings/' + meeting.id + '/participants/' + this.username;
      axios
          .get('api/participants/' + this.username)
          .then(response => axios.delete(url));
    },

    deleteMeeting(meeting) {
      this.meetings.splice(this.meetings.indexOf(meeting), 1);

      axios
          .delete('api/meetings/' + meeting.id)
          .then(response => console.log(response.data))
          .catch(response => {
            this.message = "Nie udało się usunąć spotkania, spróbuj ponownie";
          });
    },
  }
}
</script>