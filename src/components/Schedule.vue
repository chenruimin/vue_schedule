<template>
  <div>
    <Checkbox v-model="modify">Modify</Checkbox>
    <Button :disabled="!modify" type="primary" @click="handleAddScheduleButtonClick">Add schedule</Button>
    <Table :columns="columns" :data="schedules" style="width: 600px; margin: 20px auto;"></Table>
    <Modal
      v-model="modalOpen"
      :mask-closable="false"
      title="Add schedule">
      <div>
        <DatePicker
          type="daterange"
          v-model="currentSelectedDays"
          placement="bottom-end"
          placeholder="Select date"
          :options="datePickerOptions"
          style="width: 200px;"
        ></DatePicker>
      </div>
      <div style="margin-top: 10px;">
        <TimePicker
          type="timerange"
          v-model="currentSelectedTimes"
          format="HH:mm"
          :steps="[1, 15]"
          placement="bottom-end"
          placeholder="Select time"
          style="width: 200px;"
        ></TimePicker>
        <Button :disabled="!currentSelectedTimes[0]" @click="handleAddTimeClick">Add time</Button>
      </div>
      <div style="margin-top: 10px;">
        <Tag v-for="(item, i) in currentTimeSchedules" :key="i" color="green">
          {{ item.start }} - {{ item.end }}
        </Tag>
      </div>
      <div slot="footer">
        <Button type="default" size="large" @click="handleModalCancel">Cancel</Button>
        <Button type="primary" size="large" @click="handleModalOk">Submit</Button>
      </div>
    </Modal>
  </div>
</template>
<script>
  import moment from 'moment';

  export default {
    name: 'Schedule',
    data() {
      return {
        modify: false,
        modalOpen: false,
        datesOfWeek: [],
        schedules: [],
        currentTimeSchedules: [],
        currentSelectedDays: [null, null],
        currentSelectedTimes: [null, null],
        datePickerOptions: {
          disabledDate: this.isDateDisabled,
        },
        columns: [
          {
            title: 'Date',
            render: (h, { row }) => {
              if (row.days.start === row.days.end) {
                return row.days.start;
              }

              return `${row.days.start} - ${row.days.end}`;
            },
          },
          {
            title: 'Time',
            render: (h, { row }) => (
              h('div', row.times.map(item => (
                h('Tag', {
                  props: {
                    color: 'green',
                  },
                }, `${item.start}-${item.end}`)
              )))
            ),
          },
        ],
      };
    },
    methods: {
      getDatesOfWeek() {
        const days = [];
        for (let i = 0; i < 7; i += 1) {
          days.push(
            moment()
              .day(i)
              .hour(0)
              .minute(0)
              .second(0)
              .millisecond(0)
              .toDate());
        }
        return days;
      },
      handleAddScheduleButtonClick() {
        this.modalOpen = true;
      },
      handleModalOk() {
        if (!this.currentSelectedDays) {
          this.$Message.error('You must select dates!');
          return;
        }
        if (this.currentTimeSchedules.length === 0) {
          this.$Message.error('You must select any Start-End time!');
          return;
        }
        this.schedules.push({
          days: {
            start: moment(this.currentSelectedDays[0]).format('YYYY-MM-DD'),
            end: moment(this.currentSelectedDays[1]).format('YYYY-MM-DD'),
          },
          times: this.currentTimeSchedules,
        });
        this.schedules.sort((a, b) => {
          if (a.days.start < b.days.start) {
            return -1;
          }
          return 1;
        });
        this.modalOpen = false;
        this.currentSelectedDays = null;
        this.currentTimeSchedules = [];
      },
      handleModalCancel() {
        this.modalOpen = false;
      },
      handleAddTimeClick() {
        const currentSelectedTimes = {
          start: moment(this.currentSelectedTimes[0]).format('HH:mm'),
          end: moment(this.currentSelectedTimes[1]).format('HH:mm'),
        }; // format 'HH:mm'
        for (const item of this.currentTimeSchedules) {
          if (
            (item.start <= currentSelectedTimes.start
              && currentSelectedTimes.start <= item.end)
            || (item.start <= currentSelectedTimes.end
              && currentSelectedTimes.end <= item.end)
          ) {
            this.$Message.error('Time conflicts, please select time again!');
            return;
          }
        }

        this.currentTimeSchedules.push(currentSelectedTimes);

        this.currentTimeSchedules.sort((a, b) => {
          if (a.start < b.start) {
            return -1;
          }
          return 1;
        });

        this.currentSelectedTimes = [null, null];
      },
      isDateDisabled(date) {
        if (!this.datesOfWeek.find(day => (day.getTime() === date.getTime()))) {
          return true;
        }
        for (const item of this.schedules) {
          if (
            item.days.start <= moment(date).format('YYYY-MM-DD')
              && moment(date).format('YYYY-MM-DD') <= item.days.end
          ) {
            return true;
          }
        }
        return false;
      },
    },
    mounted() {
      this.datesOfWeek = this.getDatesOfWeek();
      console.log(this.datesOfWeek.map(day => day.getTime()));
    },
  };
</script>
<style scoped>

</style>
