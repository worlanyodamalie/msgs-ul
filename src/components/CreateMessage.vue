<template>
    <div>
        <DefaultLayout>  
            <form enctype="multipart/form-data"  @submit.prevent="submitSubscription">
                <p class="f4 fw8 text-green ">New Subscription</p>
        
                <li class="pt3 fw6 f4 add-message-title">Subscription Details</li>
                <div class="pv4">
                    <label class="mb2 fw5 f5 dib ">Subscription Title</label>
                    <input  class="border-input-grey br2 input-reset  pa3  db w-100" v-model="messagetitle" placeholder="Give your message a name" type="text" required  >
                </div>
                <div class="pv2 flex flex-column">
                <label class="mb2 fw6 f5 dib ">Upload Audio Message</label>
                <ul v-if="files.length" class="audio-box pv4 ph3">
                    <li v-for="file in files" :key="file.id">
                        <label class="fw5 f5 mb3 dib">{{file.name}}</label>
                        <div class="flex gap-3 pb2 items-center">
                            <audio controls :src="audiosrc"></audio>
                            <a @click="remove(file)" ><img src="@/assets/img/delete.svg" alt=""></a>
                        </div>
                        <div v-for="(segment,index) in segmenttypes"  class="pv3" :key="segment.id">
                            <label class="fw5 f5 dib mb3">Select duration for {{segment.type}} audio</label>
                            <div class="flex gap-3">
                                <div>
                                <label class="mb2 fw5 f5 dib">Start duration</label>
                                <input class="border-input-grey br2 input-reset pa2 w-100" type="number" min="0" max="60"  v-model="segmentduration[index][segment.id]['start']" >
                                </div>
                                <div>
                                <label class="mb2 fw5 f5 dib">End duration</label>
                                <input class="border-input-grey br2 input-reset pa2 w-100" type="number" min="0" max="60"  v-model="segmentduration[index][segment.id]['end']" >
                                </div>
                            </div>
                            
                        </div>
                       
                    </li>
                    
                </ul> 
                <div v-else>
                    <file-upload  class="f5 link  br2 ph3 pv3 tc white bg-light-blue w-20"
                            
                            accept="audio/*"
                            v-model="files"
                            ref="audio"
                            extensions="mp3"
                            @input-file="inputFile"> 
                            Add audio file 
                    </file-upload>
                </div>
                <!-- <input type="file" id="file" ref="audiofile" accept="audio/*" @change="handleFileUpload"/> -->
                <!-- <input type="file"  ref="audiofile" accept="audio/*" @change="selectedFile"> -->
                <!-- <button class="w-10 mt2" @click.prevent="upload.active = true">upload</button>   -->
                        
            </div>
            

            <li class="pt3 fw6 f4 add-message-title">Schedule Time</li>
            <div class="flex gap-3 pv4">
                <div>
                    <label class="mb2 fw5 f5 dib">Select Day</label>
                    <input class="border-input-grey br2 input-reset pa2 w-100" type="date" v-model="scheduledate" >
                </div>
                <div>
                    <label class="mb2 fw5 f5 dib">Select Time</label>
                    <input class="border-input-grey br2 input-reset pa2 w-100" type="time"  v-model="scheduletime" >

                </div>
            </div>
            <div class="flex pv3">
                <div >
                    <label class="mb2 fw5 f5 dib">Select Day(s) To Send</label>
                    <Multiselect
                        v-model="days"
                        mode="tags"
                        :searchable="true"
                        :createTag="true"
                        :options="options"
                        class=" pv2"
                        />
                </div>

            </div>
            <div class="flex pv3">
                <div>
                    <label class="mb2 fw5 f5 dib">Ends on</label>
                    <div class="flex gap-3 pv4">
                            <div>
                                <label class="mb2 fw5 f5 dib">Select Day</label>
                                <input class="border-input-grey br2 input-reset pa2 w-100" type="date" v-model="scheduleEnddate" >
                            </div>
                        <div>
                        <label class="mb2 fw5 f5 dib">Select Time</label>
                        <input class="border-input-grey br2 input-reset pa2 w-100" type="time"  v-model="scheduleEndtime" >

                    </div>
            </div>

                </div>
            </div>
            <div class="flex justify-end">
                <button class="border-none mt2 f5 link  br2 ph3 pv2 tc  dib white bg-green" >Schedule</button>
            </div>

            </form>
        </DefaultLayout>
            
               
    </div>
</template>
<script>
// import { ref,  watch} from 'vue'
import DefaultLayout from './Layout/DefaultLayout.vue'
import FileUpload from 'vue-upload-component'
import moment from 'moment'
import Multiselect from '@vueform/multiselect'
import '@vueform/multiselect/themes/default.css'
import axios from 'axios'

export default {
    name: 'CreateMessage',
    components: {
        FileUpload,
        Multiselect,
        DefaultLayout
    },
    data(){
       return {
           scheduledate: moment().format('YYYY-MM-DD'),
           scheduletime: moment().format('H:mm'),
           scheduleEnddate: moment().format('YYYY-MM-DD')  , 
           scheduleEndtime: moment().format('H:mm')  , 
           days: null,
           audiosrc: null,
           messagetitle: null,
           segmenttypes: [],
           segmentduration: [],
           files: [],
           attachment: null,
           options: [
               'Monday',
               'Tuesday',
               'Wednessday',
               'Thursday',
               'Friday',
               'Saturday'
           ]
       }
    } ,
    created(){
        this.getSegmentTypes()
    },
    methods: {
        handleFileUpload(){
            console.log("second file upload",this.$refs.audiofile.files[0])
        },
        inputFile(newFile,oldFile){
          if (newFile && !oldFile) {
                // Add file
                console.log("This is the new file" , newFile.file)
                newFile.blob = ''
                let URL = (window.URL || window.webkitURL)
                if (URL) {
                    newFile.blob = URL.createObjectURL(newFile.file)
                    this.audiosrc = newFile.blob
                }
                
                
                this.segmenttypes.map((segment) => {
                    let durationObj = {}
                      durationObj[segment.id] = {start: null , end: null }
                    
                    this.segmentduration.push(durationObj)
                } )

                this.attachment = newFile.file




            }
        },
        uploadaction(inputfile){
            console.log("This is the input action" , inputfile)
        },
        getSegmentTypes(){
            axios.get( 'https://sim-api.nimdee.co/markets/segment-types' , 
                        { 
                            headers: { 
                                // "Access-Control-Allow-Origin": "*", 
                                'Accept': 'application/json' ,
                                'Authorization': 'Bearer' + ' ' + localStorage.getItem("usertoken")
                                        } 
                            }

            )
            .then((response) => {
                return  this.segmenttypes = response.data.data
            })
            .catch((err) => {
                                console.log("error" , err.response)
                            })
        },
        remove(){
            this.files.pop()
            // console.log("remove" , file)
            // console.log(this.files.pop())
        },
        submitSubscription(){
            
            let formData = new FormData();
            formData.append('title' , this.messagetitle)
            this.segmentduration.map((duration,key) => {
                formData.append('market_segment_types['+ this.segmenttypes[key].id + '][start]' , duration[this.segmenttypes[key].id]['start'])
                formData.append('market_segment_types['+ this.segmenttypes[key].id + '][end]' , duration[this.segmenttypes[key].id]['end'])
               
            })
            formData.append('sent_on' , this.scheduledate)
            formData.append('sent_at' , this.scheduletime)
            formData.append('end_on' , this.scheduleEnddate)
            formData.append('end_at' , this.scheduleEndtime)
            
            this.days.map((day) => {
               formData.append('days[]' , day) 
            })
            
            formData.append('audio' , this.attachment)

            // console.log("form data" , Array.from(formData.entries()))
           
            axios.post(`${process.env.VUE_APP_API_URL}/markets/subscriptions` , 
                         formData ,
                         { 
                            headers: { 
                                'Accept': 'multipart/form-data' ,
                                'Authorization': 'Bearer' + ' ' + localStorage.getItem("usertoken")
                                        } 
                            }
                        )
                        .then((response) => {
                            if (response.status){
                                console.log("here")
                                this.$router.push({ path: '/subscription' })
                            }
                        })
                        .catch((err) => {
                            console.log(err.response)
                        })           
        }

    }
    // setup(){
    //    const upload = ref(null)

    //    let files = ref([])

    //    watch(upload,function(newUpload){
    //       console.log("Testing file" , newUpload)
    //    })

    //    return {
    //        upload,
    //        files
    //    }


    // }
   
}
</script>