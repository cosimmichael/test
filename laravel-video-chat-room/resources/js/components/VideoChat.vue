<template>
    <div>
    <div class="p-5">
        <h1 class="text-2xl mb-4">Laravel Video Chat</h1>
        <select id="video-devices" >
            <option value="0">Switch camera</option>
        </select>
        <h3>Me</h3>
        <div class="grid grid-flow-row grid-cols-3 grid-rows-3 gap-4 bg-black">
            <div id="my-video-chat-window-only"></div>
        </div>
        <h3>Others</h3>
        <div class="grid grid-flow-row grid-cols-3 grid-rows-3 gap-4 bg-black">
            <div id="my-video-chat-window"></div>
        </div>

    </div>
    </div>
</template>

<script>
export default {
    name: 'video-chat',
    data: function () {
        return {
            accessToken: '',
            participant: ''
        }
    },
    methods : {
        getAccessToken : function () {

            const _this = this
            const axios = require('axios')

            // Request a new token
            axios.get('/api/access_token')
                .then(function (response) {
                    _this.accessToken = response.data
                })
                .catch(function (error) {
                    console.log(error);
                })
                .then(function () {
                    //console.log( _this.accessToken )
                    _this.connectToRoom()
                });
        },

        connectToRoom : function () {

            const { connect, createLocalVideoTrack } = require('twilio-video');

            connect( this.accessToken, { name:'cool room' }).then(room => {

                console.log(`Successfully joined a Room: ${room}`);

                navigator.mediaDevices.enumerateDevices()
                .then(function(devices) {
                let count = 1;
                const select = document.getElementById('video-devices');
                devices.forEach(function(device) {
                    if (device.kind === 'videoinput') {
                        console.log(device.kind + ": " + device.label +
                                " id = " + device.deviceId);
                        const option = document.createElement('option');
                        option.value = device.deviceId;
                        option.innerHTML = device.label || `Camera ${count  }`;
                        select.appendChild(option);
                    }



                });
                })
                .catch(function(err) {
                console.log(err.name + ": " + err.message);
                });

                const videoChatWindow = document.getElementById('my-video-chat-window');
                const mevideoChatWindow = document.getElementById('my-video-chat-window-only');

                createLocalVideoTrack().then(track => {
                    mevideoChatWindow.appendChild(track.attach());
                });

                room.on('participantConnected', participant => {
                    console.log(`Participant "${participant.identity}" connected`);
                    this.participant = participant
                    participant.tracks.forEach(publication => {
                        if (publication.isSubscribed) {
                            const track = publication.track;
                            videoChatWindow.appendChild(track.attach());
                        }
                    });

                    participant.on('trackSubscribed', track => {
                        videoChatWindow.appendChild(track.attach());
                    });
                });

                const select = document.getElementById('video-devices');
                select.addEventListener('change', function () {
                    const select = document.getElementById('video-devices').value;
                    //alert(select);
                    const localParticipant = room.localParticipant;
                    if (select.value !== ''||select.value !== 0) {
                        createLocalVideoTrack({
                            deviceId: { exact: select.value }
                        }).then(function(localVideoTrack) {
                            console.log('this is it '+localParticipant);
                            const tracks = Array.from(localParticipant.videoTracks.values())
                            .map(publication => publication.track);
                            localParticipant.unpublishTracks(tracks);
                            tracks.forEach(function(track) {
                                track.detach();

                                mevideoChatWindow.innerHTML = '';
                                console.log('successfull');
                            });


                           // console.log(localParticipant.identity +  " removed track: " +  tracks[0].kind);



                            localParticipant.publishTrack(localVideoTrack);
                           // console.log(localParticipant.identity +  " added track: " +  localVideoTrack.kind);
                            const previewContainer = document.getElementById('my-video-chat-window-only');
                            previewContainer.appendChild(localVideoTrack.attach());


                            //this.attachTracks([localVideoTrack], previewContainer);
                        });
                    }
                });
            }, error => {
                console.error(`Unable to connect to Room: ${error.message}`);
            });
        },
        gotDevices(mediaDevices) {
            const select = document.getElementById('video-devices');
            select.innerHTML = '';
            select.appendChild(document.createElement('option'));
            let count = 1;
            mediaDevices.forEach(mediaDevice => {
                if (mediaDevice.kind === 'videoinput') {
                const option = document.createElement('option');
                option.value = mediaDevice.deviceId;
                const label = mediaDevice.label || `Camera ${count  }`;
                const textNode = document.createTextNode(label);
                option.appendChild(textNode);
                select.appendChild(option);
                }
            });
        },
        updateDevice(event) {
            const select = document.getElementById('video-devices').value;
            alert(select);
            const localParticipant = this.participant;
            if (select.value !== ''||select.value !== 0) {
                createLocalVideoTrack({
                    deviceId: { exact: select.value }
                }).then(function(localVideoTrack) {
                    const tracks = Array.from(localParticipant.videoTracks.values());
                    localParticipant.unpublishTracks(tracks);
                    log(localParticipant.identity +  " removed track: " +  tracks[0].kind);
                    detachTracks(tracks);

                    localParticipant.publishTrack(localVideoTrack);
                    log(localParticipant.identity +  " added track: " +  localVideoTrack.kind);
                    const previewContainer = document.getElementById('my-video-chat-window');
                    attachTracks([localVideoTrack], previewContainer);
                });
            }

        }
    },
    mounted : function () {
        console.log('Video chat room loading...')

        this.getAccessToken()
    }
}

</script>
<style >
    select{
        width: 200px;
        height: 30px;
    }
</style>
