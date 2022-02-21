<template>
  <div style="background-color: #f5f5f5; padding: 24px">
    <a-page-header
      :ghost="false"
      :title="nullcheck(appointment.Display)"
      :sub-title="nullcheck(appointment.Confirmation_Code)"
      @back="back"
    >
      <template slot="extra">
        <a-button
          class="startButton"
          icon="play-circle"
          :disabled="stageDisplay.start"
          @click="Timerconfirm('start')"
        >
          {{startname(appointment.Time_Status)}}
        </a-button>
        <a-button
          :disabled="stageDisplay.break"
          type="primary"
          icon="pause-circle"
          @click="Timerconfirm('break')"
        >
          Break
        </a-button>
        <a-button
          :disabled="stageDisplay.end"
          key="3"
          type="danger"
          icon="poweroff"
          @click="Timerconfirm('end')"
        >
          End
        </a-button>
      </template>
      <div class="Timer">
        <h1>{{ displaytime(service_time) }}</h1>
      </div>
    </a-page-header>
    <br />
    <a-row :gutter="[16, 8]">
      <a-col :xs="24" :lg="8" :md="12">
        <a-card :bordered="false">
          <a-row type="flex">
            <a-col flex="100px"
              ><a-icon type="play-square" class="play"
            /></a-col>
            <a-col class="datacon" padding="10px" flex="auto">
              <div>Service Start</div>
              <div class="playvalue">
                {{ nullcheck(appointment.Service_Start_Time) }}
              </div>
            </a-col>
          </a-row>
        </a-card>
      </a-col>
      <a-col :xs="24" :lg="8" :md="12">
        <a-card :bordered="false">
          <a-row type="flex">
            <a-col flex="100px"
              ><a-icon type="close-square" class="stop"
            /></a-col>
            <a-col class="datacon" padding="10px" flex="auto">
              <div>Service end</div>
              <div class="stopvalue">
                {{ nullcheck(appointment.Service_End_Time) }}
              </div>
            </a-col>
          </a-row>
        </a-card>
      </a-col>
      <a-col :xs="24" :lg="8" :md="24">
        <a-card :bordered="false">
          <a-row type="flex">
            <a-col :xs="6" :lg="6" :md="6"
              ><a-icon type="clock-circle" class="total"
            /></a-col>
            <a-col class="datacon" padding="10px" :xs="18" :lg="18" :md="18">
              <div>Total Time</div>
              <div class="totalvalue">
                {{ nullcheck(displaytimeString(service_time)) }}
              </div>
            </a-col>
          </a-row>
        </a-card>
      </a-col>
    </a-row>
    <br />
    <!-- <a-table class="tablebreak" :columns="columns" :data-source="data">
      
    </a-table> -->
    <a-table class="tablebreak" :columns="columns" :data-source="data">
    </a-table>
  </div>
</template>
<script type="text/javascript" src="https://js.zohostatic.com/creator/widgets/version/1.0/widgetsdk-min.js"></script>
<script>
const columns = [
  {
    title: "Start Break",
    dataIndex: "Break_Start",
    key: "startbreak",
  },
  {
    title: "End Break",
    dataIndex: "Break_End",
  },
  {
    title: "Duration (Min)",
    dataIndex: "Break_Duration_in_min",
  },
  {
    title: "Running Service Time (Min)",
    dataIndex: "Service_Time_in_Mins",
  },
];

var intervalObject;
var intervalSnapObject;
export default {
  data() {
    return {
      visible: false,
      data: [],
      columns,
      appointment: {},
      stageDisplay: { start: true, break: true, end: true },
      service_time: 0,
      creatorID: null,
    };
  },
  mounted() {
    console.log(window.location.href)

    __webpack_public_path__ = "/home/"
    var varStage = this;
    ZOHO.CREATOR.init().then(function (data) {
      var queryParams = ZOHO.CREATOR.UTIL.getQueryParams();
      //console.log(queryParams);
      varStage.creatorID = queryParams.appointmentID;
      var config = {
        appName: "sdtattoo",
        reportName: "Widget_Appointments",
        id: queryParams.appointmentID,
      };
      ZOHO.CREATOR.API.getRecordById(config).then(function (response) {
        //console.log(response.data);
        varStage.appointment = response.data;
        if (varStage.appointment.Time_Status == "End") {
          varStage.service_time =
            varStage.appointment.Running_Service_Time_in_minutes * 60;
          varStage.stageDisplay.start = true;
          varStage.stageDisplay.end = true;
          varStage.stageDisplay.break = true;
        } else if (varStage.appointment.Time_Status == "Break") {
          varStage.service_time =
            varStage.appointment.Running_Service_Time_in_minutes * 60;
          varStage.stageDisplay.start = false;
          varStage.stageDisplay.end = false;
          varStage.stageDisplay.break = true;
        } else if (varStage.appointment.Time_Status == "Started") {
          var d2 = new Date();
              var AddRecord = {
                data: {
                  Appointments: varStage.creatorID,
                  Break_Start: varStage.timeFormate(d2),
                },
              };
              var config3 = {
                appName: "sdtattoo",
                formName: "Break_Details",
                data: AddRecord,
              };
              ZOHO.CREATOR.API.addRecord(config3).then(function (response) {
                  varStage.BreakUpdate();
              });
          var updateRecord = {
            data: {
              Last_Break_Time: varStage.Last_Snapshot_Time,
              Time_Status: "Break",
            },
          };
          var config = {
            appName: "sdtattoo",
            reportName: "Widget_Appointments",
            id: varStage.creatorID,
            data: updateRecord,
          };
          //update record API
          ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
            //console.log("Record Updated Response");
            //console.log(response);
            varStage.appointment.Last_Break_Time = varStage.Last_Snapshot_Time;
            varStage.appointment.Time_Status = "Break";
          });
          varStage.stageDisplay.start = false;
          varStage.stageDisplay.end = true;
          varStage.stageDisplay.break = true;
          varStage.servicetime =
            varStage.appointment.Running_Service_Time_in_minutes * 60;
            varStage.BreakUpdate();
        }
         else if (!varStage.appointment.Time_Status) {
          varStage.stageDisplay.start = false;
          varStage.stageDisplay.end = true;
          varStage.stageDisplay.break = true;
        }
      });
      var cra = "(Appointments.ID ==" + varStage.creatorID + ")";
      //console.log(cra);
      var configB = {
        appName: "sdtattoo",
        reportName: "All_Break_Details",
        criteria: cra,
      };
      ZOHO.CREATOR.API.getAllRecords(configB).then(function (response) {
        //console.log(response);
        varStage.data = response.data;
      });
    });
    //Break Record
  },
  methods: {
    nullcheck(e) {
      if (e) {
        return e;
      } else {
        return "No Data";
      }
    },
    showModal() {
      this.visible = true;
    },
    hideModal() {
      this.visible = false;
    },
    timeFormate(date) {
      var mS = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "June",
        "July",
        "Aug",
        "Sept",
        "Oct",
        "Nov",
        "Dec",
      ];
      var year_s = date.getFullYear();
      var Month_s = date.getMonth();
      var Date_S = date.getDate();
      var hours = date.getHours();
      var minutes = date.getMinutes();
      var ampm = hours >= 12 ? "PM" : "AM";
      var hours = hours % 12;
      hours = hours ? hours : 12;
      minutes = minutes < 10 ? "0" + minutes : minutes;
      var strTime =
        Date_S +
        "-" +
        mS[Month_s] +
        "-" +
        year_s +
        " " +
        hours +
        ":" +
        minutes +
        " " +
        ampm;
      //console.log(strTime);
      return strTime;
    },
    servivetimemin(data) {
      return Math.ceil(data / 60);
    },
    displaytime(d) {
      d = Number(d);
      var h = ("0" + Math.floor(d / 3600)).slice(-2);
      var m = ("0" + Math.floor((d % 3600) / 60)).slice(-2);
      var s = ("0" + Math.floor((d % 3600) % 60)).slice(-2);
      return h + ":" + m + ":" + s;
    },
    displaytimeString(d) {
      d = Number(d);
      var h = ("0" + Math.floor(d / 3600)).slice(-2);
      var m = ("0" + Math.floor((d % 3600) / 60)).slice(-2);
      var s = ("0" + Math.floor((d % 3600) % 60)).slice(-2);
      return h + " Hours " + m + " Mins";
    },
    startname(stus)
    {
      if(this.appointment.Service_Start_Time)
      {
          return "Resume"
      }
      else
      {
        return "Start"
      }
    },
    back()
    {
      if(this.appointment.Time_Status =="Started")
      {
      this.$confirm({
          title: "Confirm",
          content: "Still timer is runing ?",
          okText: "Ok",
          cancelText: "Cancel",
          onOk() {
               window.parent.location.href = "https://creatorapp.zoho.com/assertiveindustriesinc/sdtattoo/#Page:Artist_Station";
          },
          onCancel() {},
        });
      }
      else
      {
        window.parent.location.href = "https://creatorapp.zoho.com/assertiveindustriesinc/sdtattoo/#Page:Artist_Station";
      }
    },
    updateBreak()
    {
      var varStage = this;
      var cra = "(Appointments.ID ==" + varStage.creatorID + ")";
      //console.log(cra);
      var configB = {
        appName: "sdtattoo",
        reportName: "All_Break_Details",
        criteria: cra,
      };
      ZOHO.CREATOR.API.getAllRecords(configB).then(function (response) {
        //console.log(response);
        varStage.data = response.data;
      });
    },
    BreakUpdate()
    {
      var varStage = this;
      var cra = "(Appointments.ID ==" + varStage.creatorID + " && Break_End == null)";
      var configB = {
        appName: "sdtattoo",
        reportName: "All_Break_Details",
        criteria: cra,
      };
      ZOHO.CREATOR.API.getAllRecords(configB).then(function (response) {
        //console.log("Break Fetch");
      //console.log(response);
      if(response.code == 3000)
      {
        if(response.data.length > 0)
        {
            var d1 = new Date();
            var updateRecord = {
              data: {
                Break_End : varStage.timeFormate(d1),
                Service_Time_in_Mins: varStage.servivetimemin(
                  varStage.service_time
                ),
              },
            };
            var config = {
              appName: "sdtattoo",
              reportName: "All_Break_Details",
              id: response.data[0].ID,
              data: updateRecord,
            };
            //update record API
            ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
              //console.log(response);
              varStage.updateBreak();
            });
        }
      }
      
      });
    },
    Timerconfirm(msgData) {
      var varStage = this;
      if (msgData == "start") {
        this.$confirm({
          title: "Confirm",
          content: "Do you want to start ?",
          okText: "Confirm",
          cancelText: "Cancel",
          onOk() {
            //console.log("start 1");
            if (!varStage.appointment.Service_Start_Time) {
              //console.log("case 1");
              var d1 = new Date();
              var updateRecord = {
                data: {
                  Last_Snapshot_Time: varStage.timeFormate(d1),
                  Service_Start_Time: varStage.timeFormate(d1),
                  Time_Status: "Started",
                  Running_Service_Time_in_minutes: 0,
                },
              };
              var config = {
                appName: "sdtattoo",
                reportName: "Widget_Appointments",
                id: varStage.creatorID,
                data: updateRecord,
              };
              //update record API
              ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
                //console.log("Record Updated Response");
                //console.log(response);
                varStage.appointment.Service_Start_Time =
                varStage.timeFormate(d1);
                varStage.appointment.Time_Status = "Started";
                varStage.appointment.Running_Service_Time_in_minutes = 0;
                varStage.appointment.Last_Snapshot_Time =
                varStage.timeFormate(d1);
              });
              intervalSnapObject = setInterval(() => {
                var d2 = new Date();
                var updateRecord2 = {
                  data: {
                    Last_Snapshot_Time: varStage.timeFormate(d1),
                    Running_Service_Time_in_minutes: varStage.servivetimemin(
                      varStage.service_time
                    ),
                  },
                };
                var config2 = {
                  appName: "sdtattoo",
                  reportName: "Widget_Appointments",
                  id: varStage.creatorID,
                  data: updateRecord2,
                };
                //update record API
                ZOHO.CREATOR.API.updateRecord(config2).then(function (
                  response
                ) {
                  //console.log("Record Updated Response I");
                  //console.log(response);
                  varStage.appointment.Running_Service_Time_in_minutes =
                    varStage.servivetimemin(varStage.service_time);
                  varStage.appointment.Last_Snapshot_Time =
                    varStage.timeFormate(d1);
                });
              }, 60000);
            } else if (
              varStage.appointment.Service_Start_Time &&
              varStage.appointment.Time_Status == "Break"
            ) {
              //console.log("case 2");

              intervalSnapObject = setInterval(() => {
                var d1 = new Date();
                var updateRecord2 = {
                  data: {
                    Last_Snapshot_Time: varStage.timeFormate(d1),
                    Running_Service_Time_in_minutes: varStage.servivetimemin(
                      varStage.service_time
                    ),
                  },
                };
                var config2 = {
                  appName: "sdtattoo",
                  reportName: "Widget_Appointments",
                  id: varStage.creatorID,
                  data: updateRecord2,
                };
                //update record API
                ZOHO.CREATOR.API.updateRecord(config2).then(function (
                  response
                ) {
                  //console.log("Record Updated Response I");
                  //console.log(response);
                  varStage.appointment.Running_Service_Time_in_minutes =
                    varStage.servivetimemin(varStage.service_time);
                  varStage.appointment.Last_Snapshot_Time =
                    varStage.timeFormate(d1);
                });
              }, 60000);
              var d1 = new Date();
              var updateRecord = {
                data: {
                  Last_Snapshot_Time: varStage.timeFormate(d1),
                  Time_Status: "Started",
                },
              };
              var config = {
                appName: "sdtattoo",
                reportName: "Widget_Appointments",
                id: varStage.creatorID,
                data: updateRecord,
              };
              //update record API
              ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
                //console.log("Record Updated Response");
                //console.log(response);
                varStage.appointment.Time_Status = "Started";
                varStage.appointment.Last_Snapshot_Time =
                  varStage.timeFormate(d1);
              });
              if (varStage.appointment.Time_Status == "Break") {
              varStage.BreakUpdate();
            }
              }
            intervalObject = setInterval(() => {
              varStage.service_time = varStage.service_time + 1;
            }, 1000);
            varStage.stageDisplay.start = true;
            varStage.stageDisplay.end = false;
            varStage.stageDisplay.break = false;
            varStage.BreakUpdate();
          },
          onCancel() {},
        });
} 
      else if (msgData == "break") {
        this.$confirm({
          title: "Confirm",
          content: "Do you want to Break ?",
          okText: "Confirm",
          cancelText: "Cancel",
          onOk() {
            var d1 = new Date();
            var updateRecord = {
              data: {
                Last_Snapshot_Time: varStage.timeFormate(d1),
                Last_Break_Time: varStage.timeFormate(d1),
                Time_Status: "Break",
                Running_Service_Time_in_minutes: varStage.servivetimemin(
                  varStage.service_time
                ),
              },
            };
            var config = {
              appName: "sdtattoo",
              reportName: "Widget_Appointments",
              id: varStage.creatorID,
              data: updateRecord,
            };
            //update record API
            ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
              //console.log("Record Updated Response");
              //console.log(response);
              varStage.appointment.Last_Break_Time = varStage.timeFormate(d1);
              varStage.appointment.Time_Status = "Break";
              varStage.Running_Service_Time_in_minutes =
                varStage.servivetimemin(varStage.service_time);
              varStage.appointment.Last_Snapshot_Time =
                varStage.timeFormate(d1);
            });
            clearInterval(intervalObject);
            clearInterval(intervalSnapObject);
            varStage.stageDisplay.start = false;
            varStage.stageDisplay.end = false;
            varStage.stageDisplay.break = true;
            //Update Break to Creator 
              var d2 = new Date();
              var AddRecord = {
                data: {
                  Appointments: varStage.creatorID,
                  Break_Start: varStage.timeFormate(d2),
                },
              };
              var config3 = {
                appName: "sdtattoo",
                formName: "Break_Details",
                data: AddRecord,
              };
              ZOHO.CREATOR.API.addRecord(config3).then(function (response) {
                  varStage.updateBreak();
              });
          },
          onCancel() {},
        });
      }
      if (msgData == "end") {
        this.$confirm({
          title: "Confirm",
          content: "Do you want to End ?",
          okText: "Confirm",
          cancelText: "Cancel",
          onOk() {
            if (varStage.appointment.Time_Status == "Break") {
              varStage.BreakUpdate();
            }
            var d1 = new Date();
            var updateRecord = {
              data: {
                Last_Snapshot_Time: varStage.timeFormate(d1),
                Service_End_Time: varStage.timeFormate(d1),
                Time_Status: "End",
                Status: "Service Completed",
                Running_Service_Time_in_minutes: varStage.servivetimemin(
                  varStage.service_time
                ),
              },
            };
            var config = {
              appName: "sdtattoo",
              reportName: "Widget_Appointments",
              id: varStage.creatorID,
              data: updateRecord,
            };
            //update record API
            ZOHO.CREATOR.API.updateRecord(config).then(function (response) {
              //console.log("Record Updated Response");
              //console.log(response);
              varStage.appointment.Service_End_Time = varStage.timeFormate(d1);
              varStage.appointment.Time_Status = "End";
              varStage.Running_Service_Time_in_minutes =
                varStage.servivetimemin(varStage.service_time);
              varStage.appointment.Last_Snapshot_Time =
                varStage.timeFormate(d1);
                  window.parent.location.href = "https://creatorapp.zoho.com/assertiveindustriesinc/sdtattoo/#Page:Artist_Station";
            });
            clearInterval(intervalObject);
            clearInterval(intervalSnapObject);
            varStage.stageDisplay.start = true;
            varStage.stageDisplay.end = true;
            varStage.stageDisplay.break = true;
          },
          onCancel() {},
        });
      }
    },
  },
};
</script>
<style>
.Timer {
  font-size: 30px;
  text-align: center;
}
.datacon {
  margin: auto;
  flex: 1 1 auto;
  padding-left: 20px;
}

.startButton {
  background-color: #52c41a !important;
  border-color: #52c41a !important;
  color: #fff !important;
}
.startButton:disabled {
  background-color: #f5f5f5 !important;
  border-color: #d9d9d9 !important;
  color: rgba(0, 0, 0, 0.25) !important;
}
tr:last-child td {
  padding-bottom: 0;
}
.tablebreak {
  background: #fff !important;
}
.play {
  text-align: center;
  font-size: 60px;
  color: green;
  background: #aaffaa;
  border-radius: 10px;
  padding: 10px;
}
.playvalue {
  font-weight: 700;
  font-size: 19px;
  color: green;
}
.stop {
  text-align: center;
  font-size: 60px;
  color: rgb(151, 0, 0);
  background: #ffaaaa;
  border-radius: 10px;
  padding: 10px;
}
.stopvalue {
  font-weight: 700;
  font-size: 19px;
  color: rgb(151, 0, 0);
}
.total {
  text-align: center;
  font-size: 60px;
  color: rgb(0, 10, 151);
  background: rgba(0, 10, 151, 0.432);
  border-radius: 10px;
  padding: 10px;
}
.totalvalue {
  font-weight: 700;
  font-size: 19px;
  color: rgb(0, 10, 151);
}
</style>