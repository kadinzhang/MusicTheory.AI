<template>
    <div class="container secondaryBackground">
        <div class="header">
            <p>
                <span>
                    <v-icon color="primary">queue_music</v-icon>
                </span>
                <router-link style="text-decoration: none" to="/">MusicTheory.ai</router-link>
            </p>
        </div>
        <div
            class="player background"
            id="player"
            v-ripple
            @click="togglePlayer"
            @mouseenter="playerHovered = true"
            @mouseleave="playerHovered = false"
        >
            <transition name="fade">
                <div
                    v-if="playerHovered && canvasLoaded"
                    id="shader"
                    @mouseover="playerHovered = true"
                ></div>
            </transition>
            <div id="loaded">
                <div class="introduction">
                    <input type="file" id="file-input" @click="rotate" title=" ">
                    <v-btn
                        outline
                        large
                        fab
                        color="primary"
                        class="uploadButton"
                        id="uploadButton"
                        tabindex="0"
                    >
                        <v-icon>add</v-icon>
                    </v-btn>
                </div>
            </div>

            <div class="visualizerLoader" id="visualizerLoader">
                <p>
                    Transcribing
                    <span class="secondary--text">{{fileName}}</span>
                    <br>
                    <span
                        class="cpuWarning"
                    >Please wait while your neural networks are hard at work!</span>
                </p>
            </div>
            <div class="canvasWrap" id="canvasWrap">
                <canvas id="canvas"></canvas>
            </div>
            <div id="controlWrap">
                <transition name="fade">
                    <v-btn
                        v-if="playerHovered"
                        @mouseover="playerHovered = true"
                        id="playButton"
                        fab
                        dark
                        large
                        outline
                        color="secondary"
                    >
                        <v-icon dark>
                            {{ //Nested ternaries are cool
                            ["unstarted", "paused", "done"].includes(this.playerState)
                            ? (this.playerState === "done" ? "replay" : "play_arrow")
                            : "pause"
                            }}
                        </v-icon>
                    </v-btn>
                </transition>
                <transition name="fade">
                    <p v-if="playerHovered" @mouseover="playerHovered = true">{{fileName}}</p>
                </transition>
            </div>
        </div>
        <div class="description">
            <p class="p1">
                Upload an
                <span class="secondary--text">audio file</span> to get started!
            </p>
            <p
                class="p2"
                style="display: none; opacity: 0"
            >Please allow about half the duration of the audio to transcribe...</p>
        </div>

        <div class="control">
            <div class="params">
                <v-text-field
                    dark
                    class="temperature"
                    v-model="temperature"
                    label="Temperature"
                    background-color="#2f3d46"
                    box
                ></v-text-field>
                <v-text-field
                    dark
                    class="steps"
                    box
                    v-model="steps"
                    label="Steps"
                    background-color="#2f3d46"
                ></v-text-field>
                <v-text-field
                    dark
                    class="chordProgression"
                    v-model="chordProgression"
                    label="Chord Progression"
                    background-color="#2f3d46"
                    box
                ></v-text-field>
            </div>
            <div style="clear:both"></div>
            <div class="submit">
                <v-btn
                    class="generate-button"
                    outline
                    color="primary"
                    @click="generateMusic"
                >Generate Music!</v-btn>
            </div>
        </div>
    </div>
</template>

<script>
import Loader from "./ui/Loader";
import { setTimeout } from "timers";
import { TweenMax } from "gsap";
import * as mm from "@magenta/music";
export default {
    name: "Transcribe",
    components: {
        Loader
    },
    data() {
        return {
            modelReady: false,
            transcriptionModel: null,
            generationModel: null,
            fileName: "",
            loader: null,
            noteSequence: null, //Transcribed music turned into magenta.js NS
            newNoteSequence: null,
            visualizer: null, //Draws noteSequence
            player: null, //Plays noteSequence
            canvasLoaded: false,
            playerHovered: false,
            playerState: "unstarted",
            audioDuration: "",
            rotated: 90,
            //Music generation options
            steps: 500,
            temperature: 1,
            chordProgression: "I, vi, IV, V, I"
        };
    },
    mounted: function() {
        var tl = new TimelineMax({
            onComplete: () => {
                this.transcriptionModel = new mm.OnsetsAndFrames(
                    "https://storage.googleapis.com/magentadata/js/checkpoints/transcription/onsets_frames_uni"
                );
                this.generationModel = new mm.MusicRNN(
                    "https://storage.googleapis.com/magentadata/js/checkpoints/music_rnn/basic_rnn"
                );
                this.initUI();
            }
        });
        tl.from(".container", 0.5, {
            opacity: 0,
            ease: Expo.easeOut
        })
            .from(".player", 0.5, {
                opacity: 0
            })
            .from(".header", 0.5, {
                opacity: 0
            })
            .from(".p1", 0.5, {
                opacity: 0
            });
    },
    methods: {
        initUI() {
            //Handle audio input
            const fileInput = document.getElementById("file-input");
            fileInput.addEventListener("change", e => {
                this.fileName = e.target.files[0].name;

                //ANIMATION SEQUENCE for after file is uploaded
                var tl = new TimelineMax({
                    onComplete: () => {
                        this.transcriptionModel.initialize().then(() => {
                            this.modelReady = true;
                            this.transcribeFile(e.target.files[0]);
                        });
                    }
                });
                tl.to("#loaded", 0.6, {
                    opacity: 0,
                    y: 20,
                    display: "none"
                })
                    .to("#visualizerLoader", 0.6, {
                        display: "block",
                        opacity: 1,
                        y: 0
                    })
                    .to(
                        ".p1",
                        0.5,
                        {
                            opacity: 0,
                            display: "none"
                        },
                        "-=0.5"
                    )
                    .to(".p2", 0.5, {
                        display: "block",
                        opacity: 1
                    });
            });
        },
        async transcribeFile(file) {
            //Transcribe the music
            await this.transcriptionModel
                .transcribeFromAudioFile(file)
                .then(noteSequence => {
                    this.noteSequence = mm.sequences.quantizeNoteSequence(
                        noteSequence,
                        8
                    );

                    //ANIMATION SEQUENCE for after music is transcribed
                    var tl = new TimelineMax();
                    tl.to("#visualizerLoader", 0.5, {
                        opacity: 0,
                        y: 20,
                        display: "none"
                    })
                        .to(
                            ".description",
                            0.5,
                            {
                                opacity: 0,
                                y: 20,
                                display: "none"
                            },
                            "-=0.25"
                        )
                        .to("#canvasWrap", 0.5, {
                            display: "block",
                            opacity: 1,
                            y: 0
                        })
                        .to("#controlWrap", 0.5, {
                            display: "block",
                            opacity: 1
                        })
                        .to(
                            ".container",
                            0.5,
                            {
                                maxHeight: "710px"
                            },
                            "-=0.25"
                        )
                        .to(
                            ".control",
                            0.5,
                            {
                                display: "block",
                                opacity: 1
                            },
                            "-=0.25"
                        );

                    //Get properties
                    this.audioDuration = this.noteSequence.timeSignatures;
                    //Setup note visualizer
                    const config = {
                        noteHeight: 10,
                        pixelsPerTimeStep: 5,
                        noteSpacing: 2,
                        noteRGB: "234, 234, 236",
                        activeNoteRGB: "52, 201, 178"
                    };
                    this.visualizer = new mm.PianoRollCanvasVisualizer(
                        this.noteSequence,
                        document.getElementById("canvas"),
                        config
                    );
                    this.canvasLoaded = true;
                    //Setup note player
                    this.player = new mm.Player();
                    this.player = new mm.SoundFontPlayer(
                        "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus"
                    );
                    this.player.loadSamples(this.noteSequence);
                    this.player.callbackObject = {
                        run: note => this.visualizer.redraw(note),
                        stop: () => {
                            console.log("done");
                            this.playerState = "done";
                        }
                    };
                });
        },
        togglePlayer() {
            //Handles player click
            if (this.canvasLoaded === true) {
                if (
                    //Player not started
                    this.playerState === "unstarted" ||
                    this.playerState === "done"
                ) {
                    this.player.resumeContext();
                    this.player.start(this.noteSequence);
                    this.playerState = "playing";
                } else if (this.playerState === "playing") {
                    //Player playing
                    this.player.pause();
                    this.playerState = "paused";
                } else if (this.playerState === "paused") {
                    //Player paused
                    this.player.resume();
                    this.playerState = "playing";
                }
            }
        },
        async generateMusic() {
            //Initialize generation model and generate music, switch visualizer
            await this.generationModel.initialize().then(() => {
                this.generationModel
                    .continueSequence(
                        this.noteSequence,
                        this.steps,
                        this.temperature
                    )
                    .then(newSequence => {
                        this.noteSequence = newSequence;
                        console.log("done");
                        const config = {
                            noteHeight: 10,
                            pixelsPerTimeStep: 5,
                            noteSpacing: 2,
                            noteRGB: "234, 234, 236",
                            activeNoteRGB: "184, 54, 200"
                        };
                        this.visualizer = new mm.PianoRollCanvasVisualizer(
                            newSequence,
                            document.getElementById("canvas"),
                            config
                        );
                        this.player.loadSamples(this.noteSequence);
                    });
            });
        },
        rotate() {
            //Simple animation on upload button click
            document.getElementById("uploadButton").style.transform =
                "rotate(" + this.rotated.toString() + "deg)";
            this.rotated += 90;
        }
    }
};
</script>

<style lang="scss" scoped>
* {
    color: white;
}
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
}
.container {
    margin-top: 50px;
    margin-bottom: 75px;
    width: 1000px;
    max-height: 510px;
    border-radius: 5px;
    padding: 0;
    opacity: 1;
    transform-origin: top;
    .description {
        padding: 27px;
        text-align: center;
        p {
            font-size: 16px;
            font-family: "Open Sans", sans-serif;
            font-weight: 200;
            opacity: 0.95;
            margin-bottom: 0;
        }
    }
    .header {
        text-align: center;
        height: 80px;
        padding: 15px;
        font-weight: 200;
        p {
            font-size: 20px;
            margin-top: 8px;
            i {
                font-size: 28px;
            }
        }
    }
    #file-input:hover .description {
        display: none !important;
    }
    .player {
        // background: #364c58;
        // background: #2b3f49;
        // background: #37474f;
        transform-origin: top;
        width: 100%;
        height: 350px;
        padding: 40px 0;
        position: relative;
        transition: 0.25s;
        &:hover {
            cursor: pointer;
        }
        #shader {
            width: 100%;
            height: 100%;
            margin-top: -40px;
            position: absolute;
            z-index: 2;
            background: rgba(50, 56, 58, 0.7);
        }
        #loaded {
            text-align: center;
            div,
            p {
                margin: 20px;
            }
            .introduction {
                input[type="file"] {
                    margin-left: -405px;
                    z-index: 3;
                    opacity: 0;
                    width: 900px;
                    height: 300px;
                    margin-top: -40px;
                    position: absolute;
                    color: transparent;
                    font-size: 0;
                    &:hover {
                        cursor: pointer;
                    }
                }
                text-align: center;
                font-size: 20px;
                width: 60%;
                margin: auto;
                font-weight: 300;
                p {
                    font-size: 16px;
                    line-height: 5px;
                    padding-top: 20px;
                    font-weight: 300;
                }
                .uploadButton {
                    margin-top: 97px;
                    i {
                        font-size: 50px;
                        transform-origin: 50% 50%;
                        transition: 0.5s;
                    }
                }
            }
        }
        .visualizerLoader {
            text-align: center;
            opacity: 0;
            transform: translate(0, 20px);
            display: none;
            p {
                font-size: 19px;
                margin-top: 95px;
                font-weight: 300;
                line-height: 35px;
                .cpuWarning {
                    font-size: 17px;
                    opacity: 0.8;
                    font-weight: 200;
                }
            }
        }
        .canvasWrap {
            width: 90%;
            transform: translate(0, 20px);
            height: 100%;
            margin: auto;
            display: none;
            opacity: 0;
            position: relative;
            #canvas {
                display: block;
                width: 100% !important;
                // image-rendering: pixelated;
                height: 100% !important;
                color: #764b7c;
            }
        }
        #controlWrap {
            display: none;
            height: 100%;
            width: 100%;
            margin-top: -270px;
            opacity: 0;
            #playButton {
                position: absolute;
                left: 50%;
                top: 50%;
                transform: translate(-50%, -50%);
                z-index: 100;
                background: rgba(74, 218, 210, 0.1) !important;
                i {
                    font-size: 40px;
                }
            }
            p {
                position: absolute;
                bottom: 50px;
                width: 100%;
                margin-left: 0px;
                text-align: center;
                z-index: 10000 !important;
                font-size: 20px;
                font-weight: 400;
                color: rgba(255, 255, 255, 0.74);
            }
        }
    }
    .control {
        display: none;
        opacity: 0;
        .flex {
            align-items: center;
            margin: auto;
        }
        .params {
            width: 100%;
            margin: auto;
            padding: 30px;
            text-align: center;
            display: flex;
            align-content: center;
            .temperature {
                width: 70px;
                margin: 20px;
            }
            .steps {
                display: inline;
                width: 70px;
                margin: 20px;
            }
            .chordProgression {
                width: 220px;
                margin: 20px;
            }
        }
        .submit {
            width: 100%;
            display: flex;
            align-content: center;
            .generate-button {
                float: none;
                width: 200px;
                margin: auto;
                // text-align: center;
                height: 60px;
                margin-top: -25px;
            }
        }
    }
}
</style>