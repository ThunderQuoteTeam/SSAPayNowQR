<template>
  <div id="app">
      <div v-if="!inMobile" class="thunderquote-footer full-width">
        <div class="m-2" >Made by <a class="no-style-link" target="_blank" href="https://thunderquote.com">ThunderQuote</a></div>
      </div>

      <div v-if="!inMobile" class="github-header full-width">
        <a class="github-corner" :href="gitHubLink" title="GitHub">
          <svg width="80" height="80" viewBox="0 0 250 250">
            <title>Github</title>
            <path d="M0 0h250v250"></path>
            <path class="octo-arm" d="M127.4 110c-14.6-9.2-9.4-19.5-9.4-19.5 3-7 1.5-11 1.5-11-1-6.2 3-2 3-2 4 4.7 2 11 2 11-2.2 10.4 5 14.8 9 16.2" fill="currentColor" style="transform-origin:130px 110px">
            </path>
            <path class="octo-body" d="M113.2 114.3s3.6 1.6 4.7.6l15-13.7c3-2.4 6-3 8.2-2.7-8-11.2-14-25 3-41 4.7-4.4 10.6-6.4 16.2-6.4.6-1.6 3.6-7.3 11.8-10.7 0 0 4.5 2.7 6.8 16.5 4.3 2.7 8.3 6 12 9.8 3.3 3.5 6.7 8 8.6 12.3 14 3 16.8 8 16.8 8-3.4 8-9.4 11-11.4 11 0 5.8-2.3 11-7.5 15.5-16.4 16-30 9-40 .2 0 3-1 7-5.2 11l-13.3 11c-1 1 .5 5.3.8 5z" fill="currentColor"></path>
            </svg>
        </a>
      </div>
     
      <div>
        <img height="300" width="300" id="logoimg" class="d-none">
      </div>
      <div>
        <img height="300" width="300" id="bgimg" class="d-none">
      </div>
      <div class="d-none">
        <canvas id="textCanvas" width="500" height="500"></canvas>
        <code v-if="false">
          <b-form-textarea id="textarea-code-output" class="output-text" :rows="5" readonly v-model="output" no-resize/>
          <b-form-textarea id="textarea-code-output" class="output-text" :rows="5" readonly v-model="output" no-resize/>
        </code>
      </div>
      
      <div class="main-container" :class="{'color-white': inMobile}">
        <div v-if="inMobile" class="d-flex justify-content-center flex-column mt-0 mb-4 mx-3">
          <div class="d-flex justify-content-center align-items-center mb-0 my-4 flex-column">
            
            <div class="mt-3 logos-box">
              <div class="d-flex flex-wrap justify-content-center">
                <a target="_blank" v-for="(logo, index) in logos" :key="'logo_' + index" :href="logo.link">
                  <img class="mx-3 header-logo" :src="logo.logo"/>
                </a>
              </div>
            </div>
            <h3 class="text-center mb-0 mt-2">Church PayNow QR Code Generator
(BSS-HIA-TQ)</h3>
            
          </div>
          <div class="main-box full-width full-height">
            <b-row class="col-row" :class="{'mx-0': inMobile}">
                <b-col sm="12" lg="6" xl="6" class="col-box col-box-first p-4" :class="{'corners-first': !inMobile, 'corners-mobile-first': inMobile}">
                  <h5 class="settings-section">General</h5>
                  <div class="mt-4">
                    <div>
                      <b-form-group
                        label-cols-sm="4"
                        label-cols-lg="3"
                        label="UEN:"
                        label-for="uen"
                        label-size="sm"
                        :state="qrForm.uen ? null : false"
                      >
                        <b-form-input placeholder="Enter UEN of recipient organisation" :debounce="debouncer" id="uen" size="sm" v-model="qrForm.uen"></b-form-input>
                      

                        <template slot="invalid-feedback">
                          <strong class="text-white">Required.</strong>
                        </template>

                      </b-form-group>
                    </div>
                    <div v-if="qrForm.uen" id="rest-of-form">
                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Editable:"
                          label-for="editable"
                          label-size="sm"
                        >
                          <b-form-radio-group
                            size="sm"
                            id="editable"
                            v-model="qrForm.editable"
                            :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                          ></b-form-radio-group>
                          <template slot="description">
                            <p class="text-white">Whether or not to allow donor to edit payment amount.</p>
                          </template>

                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Amount:"
                          label-for="amount"
                          label-size="sm"
                        >
                          <b-form-input :debounce="debouncer" id="amount" size="sm" v-model.lazy="qrForm.amount" v-money="money"></b-form-input>
                          <template slot="description">
                            <p class="text-white">Amount for donor to pay.</p>
                          </template>
                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Expiry date:"
                          label-for="expiry"
                          label-size="sm"
                        >
                          <b-form-datepicker label-no-date-selected="No expiry date set" id="expiry" size="sm" v-model="rawDate"></b-form-datepicker>
                          <b-button v-if="rawDate" @click="rawDate = ''" variant="light" size="sm" class="mt-2" block>Remove expiry date</b-button>
                          <template slot="description">
                            <p class="text-white">Set an expiry date for the PayNow QR code. If omitted, defaults to 5 years from current time.</p>
                          </template>
                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Reference Number:"
                          label-for="refnumber"
                          label-size="sm"
                        >
                          <b-form-input :debounce="debouncer" id="refnumber" size="sm" v-model="qrForm.refNumber"></b-form-input>
                          <template slot="description">
                            <p class="text-white">Reference number for Paynow Transaction. Useful if you need to track payments for recouncilation.</p>
                          </template>
                        </b-form-group>
                      </div>


                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Company:"
                          label-for="company"
                          label-size="sm"
                        >
                          <b-form-input :debounce="debouncer" id="company" size="sm" v-model="qrForm.company"></b-form-input>
                          <template slot="description">
                            <p class="text-white">Company name to embed in the QR code. Optional.</p>
                          </template>
                        </b-form-group>
                      </div>

                      <hr class="border-dim-white" />
                      <h5 class="settings-section mb-3">Logo</h5>
                      
                      

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Use custom logo:"
                            label-for="custom-logo"
                            label-size="sm"
                          >
                            <b-form-radio-group
                              size="sm"
                              id="custom-logo"
                              v-model="customLogo"
                              :options="[{text: 'Yes', value: true}, {text: 'No', value: false}, {text:'Input Text', value:'text'}]"
                            ></b-form-radio-group>
                          </b-form-group>
                        </div>
                      <div>

                        <div>
                          
                          <div v-if="customLogo === 'text'">
                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Text:"
                                label-for="logo-text"
                                label-size="sm"
                                class="my-2"
                              >
                                <b-textarea id="logo-text" v-model="qrLogoText.text"></b-textarea>
                              </b-form-group>
                            </div>

                            <div class="d-flex justify-content-end mb-2">
                                <b-button v-b-toggle.text-advanced size="sm" variant="light">More Text Settings</b-button>
                            </div>

                            <b-collapse id="text-advanced" class="mt-2">
                                <b-card body-class="col-box-first">
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Font Size:"
                                      label-for="logo-text-font-size"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-font-size" size="sm" type="range" step="1" min="0" max="200" v-model="qrLogoText.fontSize"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Font Weight:"
                                      label-for="text-font-weight"
                                      label-size="sm"
                                    >
                                      <b-form-radio-group
                                        size="sm"
                                        id="text-font-weight"
                                        v-model="customLogo"
                                        :options="[{text: 'Normal', value: 'normal'}, {text: 'Bold', value: 'bold'}]"
                                      ></b-form-radio-group>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text color:"
                                      label-for="text-logo-color"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" ref="logoTextColor" id="text-logo-color" size="sm" type="color" v-model="qrLogoText.textColor"></b-form-input>
                                    </b-form-group>
                                  </div>
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Position X:"
                                      label-for="logo-text-x"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-x" size="sm" type="range" step="1" min="25" max="500" v-model="qrLogoText.x"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Position Y:"
                                      label-for="logo-text-y"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-y" size="sm" type="range" step="1" min="-23" max="500" v-model="qrLogoText.y"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Width:"
                                      label-for="logo-text-width"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-width" size="sm" type="range" step="1" min="0" max="450" v-model="qrLogoText.width"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Height:"
                                      label-for="logo-text-height"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-height" size="sm" type="range" step="1" min="0" max="500" v-model="qrLogoText.height"></b-form-input>
                                    </b-form-group>
                                  </div>
                                  
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Line Height:"
                                      label-for="logo-text-line-height"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-line-height" size="sm" type="range" step="1" min="90" max="300" v-model="qrLogoText.lineHeight"></b-form-input>
                                    </b-form-group>
                                  </div>
                                </b-card>
                              </b-collapse>
                          </div>  

                           <div v-if="qrLogo.src">
                            
                            <div v-if="false">
                              <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Logo height:"
                                  label-for="logo-height"
                                  label-size="sm"
                                  class="mt-2"
                                >
                                  <b-form-input :debounce="debouncer" id="logo-height" size="sm" type="range" step="1" min="1" max="121" v-model="qrLogo.logoHeight"></b-form-input>
                                </b-form-group>
                              </div>

                              <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Logo width:"
                                  label-for="logo-width"
                                  label-size="sm"
                                  class="mt-2"
                                >
                                  <b-form-input :debounce="debouncer" id="logo-width" size="sm" type="range" step="1" min="1" max="121" v-model="qrLogo.logoWidth"></b-form-input>
                                </b-form-group>
                              </div>
                            </div>

                            <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Logo size:"
                                  label-for="logo-width"
                                  label-size="sm"
                                  class="mt-2"
                                >
                                  <b-form-input :debounce="debouncer" id="logo-width" size="sm" type="range" step="1" min="1" max="120" v-model="qrLogo.logoHeightWidth"></b-form-input>
                                </b-form-group>
                              </div>

                            

                              

                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Transparent logo background:"
                                label-for="logo-background-transparent"
                                label-size="sm"
                              >
                                <b-form-radio-group
                                  size="sm"
                                  id="logo-background-transparent"
                                  v-model="qrLogo.bgTransparent"
                                  :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                                ></b-form-radio-group>
                              </b-form-group>
                            </div>
                            <div v-if="!qrLogo.bgTransparent">
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Logo background color:"
                                label-for="logo-bg-color"
                                label-size="sm"
                                class="mt-2"
                              >
                                <b-form-input :debounce="debouncer" ref="logoBgColor" id="logo-bg-color" size="sm" type="color" v-model="qrLogo.bgColor"></b-form-input>
                              </b-form-group>
                            </div>
                          
                          </div>

                          <div v-else-if="customLogo === true">
                            <b-form-group
                              label-cols-sm="4"
                              label-cols-lg="3"
                              label="Image:"
                              label-for="logo"
                              label-size="sm"
                            >
                              <b-form-file
                                @input="loadLogoImage"
                                v-model="qrLogo.src"
                                placeholder="QR Code logo image"
                                drop-placeholder="Drop file here..."
                                size="sm"
                                id="logo"
                                accept="image/*"
                                class="mb-3"
                              ></b-form-file>
                            </b-form-group>

                            <div v-if="qrLogo && qrLogo.src" class="d-flex justify-content-between align-items-center">
                              <small class="text-italic">
                                {{'Selected logo image: ' + qrLogo.src.name}}
                              </small>
                              <b-button variant="light" class="mt-2 mb-3" size="sm" @click="removeLogo">Remove</b-button>
                            </div>
                          </div>
                        </div>
                      
                        

                        
                      </div>

                      <hr class="border-dim-white" />
                      <h5 class="settings-section mb-3">General QR Code Image Settings</h5>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="QR Code light color:"
                          label-for="qrcode-light-color"
                          label-size="sm"
                          class="mt-2"
                        >
                          <b-form-input :debounce="debouncer" id="qrcode-light-color" size="sm" type="color" v-model="qr.color.light"></b-form-input>
                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="QR Code dark color:"
                          label-for="qrcode-dark-color"
                          label-size="sm"
                          class="mt-2"
                        >
                          <b-form-input :debounce="debouncer" id="qrcode-dark-color" size="sm" type="color" v-model="qr.color.dark"></b-form-input>
                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="QR Code Border Size:"
                          label-for="border-size"
                          label-size="sm"
                          class="mt-2"
                        >
                          <b-form-input :debounce="debouncer" id="border-size" size="sm" type="range" step="1" min="12" max="20" v-model="qr.border.size"></b-form-input>
                        </b-form-group>
                      </div>

                      <div v-if="qrLogo.src">
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="QR Code Border Color:"
                          label-for="qrcode-border-color"
                          label-size="sm"
                          class="mt-2"
                        >
                          <b-form-input :debounce="debouncer" id="qrcode-border-color" size="sm" type="color" v-model="qr.border.color"></b-form-input>
                        </b-form-group>
                      </div>

                      <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="Image:"
                          label-for="logo"
                          label-size="sm"
                        >
                        <b-form-file
                          @input="loadBgImage"
                          v-model="qrBg.src"
                          placeholder="QR Code background image"
                          drop-placeholder="Drop file here..."
                          size="sm"
                          id="logo"
                          accept="image/*"
                          class="mb-3"
                        ></b-form-file>
                        </b-form-group>

                        <div v-if="qrBg.src">
                          <div class="d-flex justify-content-between align-items-center">
                            <small class="text-italic">
                              {{'Selected Background Image: ' + qrBg.src.name}}
                            </small>
                            <b-button variant="light" class="mt-2 mb-3" size="sm" @click="removeBackground">Remove</b-button>

                          </div>

                          <div>
                            <b-form-group
                              label-cols-sm="4"
                              label-cols-lg="3"
                              label="Auto background color:"
                              label-for="auto-bg-color"
                              label-size="sm"
                            >
                              <b-form-radio-group
                                size="sm"
                                id="auto-bg-color"
                                v-model="qrBg.autoBgColor"
                                :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                              ></b-form-radio-group>
                            </b-form-group>
                          </div>

                          <div>
                            <b-form-group
                              label-cols-sm="4"
                              label-cols-lg="3"
                              label="Background Image Alpha:"
                              label-for="bg-img-alpha"
                              label-size="sm"
                              class="mt-2"
                            >
                              <b-form-input :debounce="debouncer" id="bg-img-alpha" size="sm" type="range" step="0.1" min="0.1" max="1.0" v-model="qrBg.bgImgAlpha"></b-form-input>
                            </b-form-group>
                          </div>
                        </div>
                      </div>
                    </div>
                    
                    <div class="mt-3 d-flex justify-content-center">
                      <b-button variant="light" @click="resetGenerator">Reset Generator</b-button>
                    </div>

                  </div>
                </b-col>
                <b-col sm="12" lg="6" xl="6" class="col-box col-box-second p-4" :class="{'corners-second': !inMobile, 'corners-mobile-second': inMobile}">
                  <div v-if="qrForm.uen" class="second-box-content-container">
                    <div v-if="false">
                      <label>QR Code Text Output (click on this box to copy):</label>
                      <div>
                        <code>
                          <b-form-textarea @click="copyToClipboard" class="output-text" :rows="5" readonly v-model="output" no-resize/>
                        </code>
                      </div>
                    </div>
                    <div class="mt-3">
                      <label>QR Code Image Output:</label>
                      <div :class="{'qr-image-container': !inMobile}" class="d-flex justify-content-center align-items-center">
                        <div v-show="!generating">
                          <div :class="{'qr-image-container': !inMobile}" id="qr-image" ref="qrcode" @click="downloadqRCodeImage" class="d-flex justify-content-center align-items-center"></div>
                        </div>
                        <div v-show="generating">
                          <p>Generating...</p>
                        </div>
                      </div>
                      <div class="mt-3">
                        <div class="text-muted tip">Tip: Try scanning the QR Code first to make sure it's readable by the PayNow QR reader before usage.</div>
                      </div>
                      <div class="mt-3 d-flex justify-content-center">
                        <b-button variant="light" @click="downloadqRCodeImage">Save QR Code Image</b-button>
                      </div>
                    </div>
                  </div>
                  <div class="d-flex flex-column full-height" v-else-if="false">
                    <welcome-message/>
                  </div>
                </b-col>
            </b-row>
          </div>
        </div>

        <div v-else>
          <div class="spacer"></div>

          <div class="d-inline">
            <div class="diagonal-box">
            </div>
            <div class="diagonal-box-2">
            </div>
          </div>

          <div>
            <div class="new-child-box py-2 px-4" :class="{'new-top': !toggleNewTop}" v-if="qrForm.uen">
              <div>
                <div class="d-flex justify-content-center mt-2">
                  <div>Generated Result Preview: </div>
                </div>
                <div class="d-none justify-content-between align-items-center">
                  <label class="mb-0">QR Code Image Output:</label>
                  <b-button variant="light" size="sm" @click="copyToClipboard">Copy text generated</b-button>
                </div>
                <div :class="{'qr-image-container': !inMobile}" class="d-flex justify-content-center align-items-center">
                  <div v-show="!generating">
                    <div :class="{'qr-image-container': !inMobile}" id="qr-image" ref="qrcode" @click="downloadqRCodeImage" class="d-flex justify-content-center align-items-center"></div>
                  </div>
                  <div v-show="generating">
                    <p>Generating...</p>
                  </div>
                </div>
                <div>
                  <div class="text-muted tip">Tip: Try scanning the QR Code first to make sure it's readable by the PayNow QR reader before usage.</div>
                </div>
                <div class="mt-3 mb-1 d-flex justify-content-center">
                  <b-button variant="light" @click="downloadqRCodeImage">Save QR Code Image</b-button>
                </div>
              </div>
            </div>
            <div class="new-box-container flex-column mb-5 d-flex justify-content-center align-items-center">
                <div class="new-box px-4 pb-4" >
                  <div class="d-flex justify-content-center align-items-center mb-0 my-4 flex-column">
                    <div class="logos-box mb-3">
                      <div class="d-flex flex-wrap justify-content-center">
                        <a target="_blank" v-for="(logo, index) in logos" :key="'logo_' + index" :href="logo.link">
                          <img class="mx-3 header-logo" :src="logo.logo"/>
                        </a>
                      </div>
                    </div>
                    <h3 class="text-center">
                      <div>Church PayNow QR Code Generator</div>
                      <div>(BSS-HIA-TQ)</div>
                    </h3>
                    
                    
                  </div>
                  <b-row class="mx-0">
                    <b-col :cols="qrForm.uen ? 7 : 12" :md="qrForm.uen ? 7 : 12" :lg="qrForm.uen ? 7 : 12" :xl="qrForm.uen ? 7 : 12">
                        <div>
                        <b-form-group
                          label-cols-sm="4"
                          label-cols-lg="3"
                          label="UEN:"
                          label-for="uen"
                          label-size="sm"
                          :state="qrForm.uen ? null : false"
                        >
                          <b-form-input placeholder="Enter UEN of recipient organisation" :debounce="debouncer" id="uen" size="sm" v-model="qrForm.uen"></b-form-input>
                         

                          <template slot="invalid-feedback">
                            <strong class="text-white">Required.</strong>
                          </template>

                        </b-form-group>
                      </div>
                      <div v-if="qrForm.uen" id="rest-of-form">
                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Editable:"
                            label-for="editable"
                            label-size="sm"
                          >
                            <b-form-radio-group
                              size="sm"
                              id="editable"
                              v-model="qrForm.editable"
                              :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                            ></b-form-radio-group>
                            <template slot="description">
                              <p class="text-white">Whether or not to allow donor to edit payment amount.</p>
                            </template>

                          </b-form-group>
                        </div>

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Amount:"
                            label-for="amount"
                            label-size="sm"
                          >
                            <b-form-input :debounce="debouncer" id="amount" size="sm" v-model.lazy="qrForm.amount" v-money="money"></b-form-input>
                            <template slot="description">
                              <p class="text-white">Amount for donor to pay.</p>
                            </template>
                          </b-form-group>
                        </div>

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Expiry date:"
                            label-for="expiry"
                            label-size="sm"
                          >
                            <b-form-datepicker label-no-date-selected="No expiry date set" id="expiry" size="sm" v-model="rawDate"></b-form-datepicker>
                            <b-button v-if="rawDate" @click="rawDate = ''" variant="light" size="sm" class="mt-2" block>Remove expiry date</b-button>
                            <template slot="description">
                              <p class="text-white">Set an expiry date for the PayNow QR code. If omitted, defaults to 5 years from current time.</p>
                            </template>
                          </b-form-group>
                        </div>

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Reference Number:"
                            label-for="refnumber"
                            label-size="sm"
                          >
                            <b-form-input :debounce="debouncer" id="refnumber" size="sm" v-model="qrForm.refNumber"></b-form-input>
                            <template slot="description">
                              <p class="text-white">Reference number for Paynow Transaction. Useful if you need to track payments for recouncilation.</p>
                            </template>
                          </b-form-group>
                        </div>


                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Company:"
                            label-for="company"
                            label-size="sm"
                          >
                            <b-form-input :debounce="debouncer" id="company" size="sm" v-model="qrForm.company"></b-form-input>
                            <template slot="description">
                              <p class="text-white">Company name to embed in the QR code. Optional.</p>
                            </template>
                          </b-form-group>
                        </div>

                  <hr class="border-dim-white" />
                        <h5 class="settings-section mb-3">Logo</h5>
                        
                        

                        <div>
                            <b-form-group
                              label-cols-sm="4"
                              label-cols-lg="3"
                              label="Use custom logo:"
                              label-for="custom-logo"
                              label-size="sm"
                            >
                              <b-form-radio-group
                                size="sm"
                                id="custom-logo"
                                v-model="customLogo"
                                :options="[{text: 'Yes', value: true}, {text: 'No', value: false},{text:'Input Text', value:'text'}]"
                              ></b-form-radio-group>
                            </b-form-group>
                          </div>
                        <div>

                          

                          <div>
                            
                            <div v-if="customLogo === 'text'">
                              <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Text:"
                                  label-for="logo-text"
                                  label-size="sm"
                                  class="my-2"
                                >
                                  <b-textarea id="logo-text" size="sm" v-model="qrLogoText.text"></b-textarea>
                                </b-form-group>
                              </div>

                              <div class="d-flex justify-content-end mb-2">
                                <b-button v-b-toggle.text-advanced size="sm" variant="light">More Text Settings</b-button>
                              </div>

                              <b-collapse id="text-advanced" class="mt-2">
                                <b-card body-class="col-box-first">
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Font Size:"
                                      label-for="logo-text-font-size"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-font-size" size="sm" type="range" step="1" min="0" max="200" v-model="qrLogoText.fontSize"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Font Weight:"
                                      label-for="text-font-weight"
                                      label-size="sm"
                                    >
                                      <b-form-radio-group
                                        size="sm"
                                        id="text-font-weight"
                                        v-model="qrLogoText.fontWeight"
                                        :options="[{text: 'Normal', value: 'normal'}, {text: 'Bold', value: 'bold'}]"
                                      ></b-form-radio-group>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text color:"
                                      label-for="text-logo-color"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" ref="logoTextColor" id="text-logo-color" size="sm" type="color" v-model="qrLogoText.textColor"></b-form-input>
                                    </b-form-group>
                                  </div>
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Position X:"
                                      label-for="logo-text-x"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-x" size="sm" type="range" step="1" min="25" max="500" v-model="qrLogoText.x"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Position Y:"
                                      label-for="logo-text-y"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-y" size="sm" type="range" step="1" min="-23" max="500" v-model="qrLogoText.y"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Width:"
                                      label-for="logo-text-width"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-width" size="sm" type="range" step="1" min="0" max="450" v-model="qrLogoText.width"></b-form-input>
                                    </b-form-group>
                                  </div>

                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Height:"
                                      label-for="logo-text-height"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-height" size="sm" type="range" step="1" min="0" max="500" v-model="qrLogoText.height"></b-form-input>
                                    </b-form-group>
                                  </div>
                                  
                                  <div>
                                    <b-form-group
                                      label-cols-sm="4"
                                      label-cols-lg="3"
                                      label="Text Line Height:"
                                      label-for="logo-text-line-height"
                                      label-size="sm"
                                      class="mt-2"
                                    >
                                      <b-form-input :debounce="debouncer" id="logo-text-line-height" size="sm" type="range" step="1" min="90" max="300" v-model="qrLogoText.lineHeight"></b-form-input>
                                    </b-form-group>
                                  </div>
                                </b-card>
                              </b-collapse>
                            </div>

                            <div v-else-if="customLogo === true">
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Image:"
                                label-for="logo"
                                label-size="sm"
                              >
                                <b-form-file
                                  @input="loadLogoImage"
                                  v-model="qrLogo.src"
                                  placeholder="QR Code logo image"
                                  drop-placeholder="Drop file here..."
                                  size="sm"
                                  id="logo"
                                  accept="image/*"
                                  class="mb-3"
                                ></b-form-file>
                              </b-form-group>

                              <div v-if="qrLogo && qrLogo.src" class="d-flex justify-content-between align-items-center">
                                <small class="text-italic">
                                  {{'Selected logo image: ' + qrLogo.src.name}}
                                </small>
                                <b-button variant="light" class="mt-2 mb-3" size="sm" @click="removeLogo">Remove</b-button>
                              </div>
                            </div>
                          </div>
                        
                          <div v-if="qrLogo.src">
                            
                            <div v-if="false">
                              <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Logo height:"
                                  label-for="logo-height"
                                  label-size="sm"
                                  class="mt-2"
                                >
                                  <b-form-input :debounce="debouncer" id="logo-height" size="sm" type="range" step="1" min="1" max="121" v-model="qrLogo.logoHeight"></b-form-input>
                                </b-form-group>
                              </div>

                              <div>
                                <b-form-group
                                  label-cols-sm="4"
                                  label-cols-lg="3"
                                  label="Logo width:"
                                  label-for="logo-width"
                                  label-size="sm"
                                  class="mt-2"
                                >
                                  <b-form-input :debounce="debouncer" id="logo-width" size="sm" type="range" step="1" min="1" max="121" v-model="qrLogo.logoWidth"></b-form-input>
                                </b-form-group>
                              </div>
                            </div>

                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Logo size:"
                                label-for="logo-size"
                                label-size="sm"
                                class="mt-2"
                              >
                                <b-form-input :debounce="debouncer" id="logo-size" size="sm" type="range" step="1" min="1" max="120" v-model="qrLogo.logoHeightWidth"></b-form-input>
                              </b-form-group>
                            </div>

                            

                              

                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Transparent logo background:"
                                label-for="logo-background-transparent"
                                label-size="sm"
                              >
                                <b-form-radio-group
                                  size="sm"
                                  id="logo-background-transparent"
                                  v-model="qrLogo.bgTransparent"
                                  :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                                ></b-form-radio-group>
                              </b-form-group>
                            </div>
                            <div v-if="!qrLogo.bgTransparent">
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Logo background color:"
                                label-for="logo-bg-color"
                                label-size="sm"
                                class="mt-2"
                              >
                                <b-form-input :debounce="debouncer" ref="logoBgColor" id="logo-bg-color" size="sm" type="color" v-model="qrLogo.bgColor"></b-form-input>
                              </b-form-group>
                            </div>
                          
                          </div>

                          
                        </div>

                        <hr class="border-dim-white" />
                        <h5 class="settings-section mb-3">General QR Code Image Settings</h5>


                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="QR Code light color:"
                            label-for="qrcode-light-color"
                            label-size="sm"
                            class="mt-2"
                          >
                            <b-form-input :debounce="debouncer" id="qrcode-light-color" size="sm" type="color" v-model="qr.color.light"></b-form-input>
                          </b-form-group>
                        </div>

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="QR Code dark color:"
                            label-for="qrcode-dark-color"
                            label-size="sm"
                            class="mt-2"
                          >
                            <b-form-input :debounce="debouncer" id="qrcode-dark-color" size="sm" type="color" v-model="qr.color.dark"></b-form-input>
                          </b-form-group>
                        </div>

                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="QR Code Border Size:"
                            label-for="border-size"
                            label-size="sm"
                            class="mt-2"
                          >
                            <b-form-input :debounce="debouncer" id="border-size" size="sm" type="range" step="1" min="12" max="20" v-model="qr.border.size"></b-form-input>
                          </b-form-group>
                        </div>

                        <div v-if="qrLogo.src">
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="QR Code Border Color:"
                            label-for="qrcode-border-color"
                            label-size="sm"
                            class="mt-2"
                          >
                            <b-form-input :debounce="debouncer" id="qrcode-border-color" size="sm" type="color" v-model="qr.border.color"></b-form-input>
                          </b-form-group>
                        </div>

                      
                        <div>
                          <b-form-group
                            label-cols-sm="4"
                            label-cols-lg="3"
                            label="Image:"
                            label-for="logo"
                            label-size="sm"
                          >
                          <b-form-file
                            @input="loadBgImage"
                            v-model="qrBg.src"
                            placeholder="QR Code background image"
                            drop-placeholder="Drop file here..."
                            size="sm"
                            id="logo"
                            accept="image/*"
                            class="mb-3"
                          ></b-form-file>
                          </b-form-group>

                          <div v-if="qrBg.src">
                            <div class="d-flex justify-content-between align-items-center">
                              <small class="text-italic">
                                {{'Selected Background Image: ' + qrBg.src.name}}
                              </small>
                              <b-button variant="light" class="mt-2 mb-3" size="sm" @click="removeBackground">Remove</b-button>

                            </div>

                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Auto background color:"
                                label-for="auto-bg-color"
                                label-size="sm"
                              >
                                <b-form-radio-group
                                  size="sm"
                                  id="auto-bg-color"
                                  v-model="qrBg.autoBgColor"
                                  :options="[{text: 'Yes', value: true}, {text: 'No', value: false}]"
                                ></b-form-radio-group>
                              </b-form-group>
                            </div>

                            <div>
                              <b-form-group
                                label-cols-sm="4"
                                label-cols-lg="3"
                                label="Background Image Alpha:"
                                label-for="bg-img-alpha"
                                label-size="sm"
                                class="mt-2"
                              >
                                <b-form-input :debounce="debouncer" id="bg-img-alpha" size="sm" type="range" step="0.1" min="0.1" max="1.0" v-model="qrBg.bgImgAlpha"></b-form-input>
                              </b-form-group>
                            </div>
                          </div>
                        </div>
                      </div>
                      <div v-if="qrForm.uen" class="my-3 d-flex justify-content-center">
                        <b-button variant="light" @click="resetGenerator">Reset Generator</b-button>
                      </div>
                    </b-col>

                    <b-col cols="6" md="5" lg="6" xl="5" class="background"></b-col>
                  </b-row>
                </div>
                <div v-if="false" class="new-box-2">
                  <welcome-message/>
                </div>
            </div>
          </div>
          
        </div>



      </div>
  </div>
</template>

<script>
import PaynowQR from 'paynowqr';
import * as QRCode from 'easyqrcodejs'   
import canvasTxt from 'canvas-txt'
import moment from 'moment'
import {VMoney} from 'v-money'
import welcomeMessage from '@/assets/components/welcomeMessage'

let getInitialData = () => {
  return {
      logos: [{
        link: 'https://thunderquote.com',
        logo: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAf4AAAH+CAYAAAB9b2wlAAAACXBIWXMAAAsTAAALEwEAmpwYAAA8f2lUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS41LWMwMjEgNzkuMTU1NzcyLCAyMDE0LzAxLzEzLTE5OjQ0OjAwICAgICAgICAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iCiAgICAgICAgICAgIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIKICAgICAgICAgICAgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIgogICAgICAgICAgICB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIKICAgICAgICAgICAgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIKICAgICAgICAgICAgeG1sbnM6dGlmZj0iaHR0cDovL25zLmFkb2JlLmNvbS90aWZmLzEuMC8iCiAgICAgICAgICAgIHhtbG5zOmV4aWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20vZXhpZi8xLjAvIj4KICAgICAgICAgPHhtcDpDcmVhdG9yVG9vbD5BZG9iZSBQaG90b3Nob3AgQ0MgMjAxNCAoV2luZG93cyk8L3htcDpDcmVhdG9yVG9vbD4KICAgICAgICAgPHhtcDpDcmVhdGVEYXRlPjIwMTctMDYtMjhUMTc6MTM6MTgrMDg6MDA8L3htcDpDcmVhdGVEYXRlPgogICAgICAgICA8eG1wOk1vZGlmeURhdGU+MjAxNy0wNy0xOFQxMTo0MTowNiswODowMDwveG1wOk1vZGlmeURhdGU+CiAgICAgICAgIDx4bXA6TWV0YWRhdGFEYXRlPjIwMTctMDctMThUMTE6NDE6MDYrMDg6MDA8L3htcDpNZXRhZGF0YURhdGU+CiAgICAgICAgIDxkYzpmb3JtYXQ+aW1hZ2UvcG5nPC9kYzpmb3JtYXQ+CiAgICAgICAgIDxwaG90b3Nob3A6Q29sb3JNb2RlPjM8L3Bob3Rvc2hvcDpDb2xvck1vZGU+CiAgICAgICAgIDxwaG90b3Nob3A6RG9jdW1lbnRBbmNlc3RvcnM+CiAgICAgICAgICAgIDxyZGY6QmFnPgogICAgICAgICAgICAgICA8cmRmOmxpPmFkb2JlOmRvY2lkOnBob3Rvc2hvcDplN2JjYmY0MS02NjExLTExZTctOTAwMC1mNjEzYmE1ZDRhOGE8L3JkZjpsaT4KICAgICAgICAgICAgPC9yZGY6QmFnPgogICAgICAgICA8L3Bob3Rvc2hvcDpEb2N1bWVudEFuY2VzdG9ycz4KICAgICAgICAgPHhtcE1NOkluc3RhbmNlSUQ+eG1wLmlpZDo0YjAxNjY3YS04ZGI2LTFmNGMtOWFhMC0wM2E0ZmZjMTNiODE8L3htcE1NOkluc3RhbmNlSUQ+CiAgICAgICAgIDx4bXBNTTpEb2N1bWVudElEPmFkb2JlOmRvY2lkOnBob3Rvc2hvcDplODEzYzk3My02YjY5LTExZTctOThlOS05YjIxMWQ0NjY0NzA8L3htcE1NOkRvY3VtZW50SUQ+CiAgICAgICAgIDx4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ+eG1wLmRpZDo3YTFkYTcyZS04NTg4LTFmNDEtYjc3OS0zYTQ2OGE2ODU5Yjc8L3htcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD4KICAgICAgICAgPHhtcE1NOkhpc3Rvcnk+CiAgICAgICAgICAgIDxyZGY6U2VxPgogICAgICAgICAgICAgICA8cmRmOmxpIHJkZjpwYXJzZVR5cGU9IlJlc291cmNlIj4KICAgICAgICAgICAgICAgICAgPHN0RXZ0OmFjdGlvbj5jcmVhdGVkPC9zdEV2dDphY3Rpb24+CiAgICAgICAgICAgICAgICAgIDxzdEV2dDppbnN0YW5jZUlEPnhtcC5paWQ6N2ExZGE3MmUtODU4OC0xZjQxLWI3NzktM2E0NjhhNjg1OWI3PC9zdEV2dDppbnN0YW5jZUlEPgogICAgICAgICAgICAgICAgICA8c3RFdnQ6d2hlbj4yMDE3LTA2LTI4VDE3OjEzOjE4KzA4OjAwPC9zdEV2dDp3aGVuPgogICAgICAgICAgICAgICAgICA8c3RFdnQ6c29mdHdhcmVBZ2VudD5BZG9iZSBQaG90b3Nob3AgQ0MgMjAxNCAoV2luZG93cyk8L3N0RXZ0OnNvZnR3YXJlQWdlbnQ+CiAgICAgICAgICAgICAgIDwvcmRmOmxpPgogICAgICAgICAgICAgICA8cmRmOmxpIHJkZjpwYXJzZVR5cGU9IlJlc291cmNlIj4KICAgICAgICAgICAgICAgICAgPHN0RXZ0OmFjdGlvbj5zYXZlZDwvc3RFdnQ6YWN0aW9uPgogICAgICAgICAgICAgICAgICA8c3RFdnQ6aW5zdGFuY2VJRD54bXAuaWlkOjMzNDc0NWEzLTBjNTktNTk0Yy04NjEzLWRjMDg0ZmVhNmMyYzwvc3RFdnQ6aW5zdGFuY2VJRD4KICAgICAgICAgICAgICAgICAgPHN0RXZ0OndoZW4+MjAxNy0wNy0xMVQxNjoyMTozMSswODowMDwvc3RFdnQ6d2hlbj4KICAgICAgICAgICAgICAgICAgPHN0RXZ0OnNvZnR3YXJlQWdlbnQ+QWRvYmUgUGhvdG9zaG9wIENDIDIwMTQgKFdpbmRvd3MpPC9zdEV2dDpzb2Z0d2FyZUFnZW50PgogICAgICAgICAgICAgICAgICA8c3RFdnQ6Y2hhbmdlZD4vPC9zdEV2dDpjaGFuZ2VkPgogICAgICAgICAgICAgICA8L3JkZjpsaT4KICAgICAgICAgICAgICAgPHJkZjpsaSByZGY6cGFyc2VUeXBlPSJSZXNvdXJjZSI+CiAgICAgICAgICAgICAgICAgIDxzdEV2dDphY3Rpb24+c2F2ZWQ8L3N0RXZ0OmFjdGlvbj4KICAgICAgICAgICAgICAgICAgPHN0RXZ0Omluc3RhbmNlSUQ+eG1wLmlpZDo0YjAxNjY3YS04ZGI2LTFmNGMtOWFhMC0wM2E0ZmZjMTNiODE8L3N0RXZ0Omluc3RhbmNlSUQ+CiAgICAgICAgICAgICAgICAgIDxzdEV2dDp3aGVuPjIwMTctMDctMThUMTE6NDE6MDYrMDg6MDA8L3N0RXZ0OndoZW4+CiAgICAgICAgICAgICAgICAgIDxzdEV2dDpzb2Z0d2FyZUFnZW50PkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE0IChXaW5kb3dzKTwvc3RFdnQ6c29mdHdhcmVBZ2VudD4KICAgICAgICAgICAgICAgICAgPHN0RXZ0OmNoYW5nZWQ+Lzwvc3RFdnQ6Y2hhbmdlZD4KICAgICAgICAgICAgICAgPC9yZGY6bGk+CiAgICAgICAgICAgIDwvcmRmOlNlcT4KICAgICAgICAgPC94bXBNTTpIaXN0b3J5PgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8dGlmZjpYUmVzb2x1dGlvbj43MjAwMDAvMTAwMDA8L3RpZmY6WFJlc29sdXRpb24+CiAgICAgICAgIDx0aWZmOllSZXNvbHV0aW9uPjcyMDAwMC8xMDAwMDwvdGlmZjpZUmVzb2x1dGlvbj4KICAgICAgICAgPHRpZmY6UmVzb2x1dGlvblVuaXQ+MjwvdGlmZjpSZXNvbHV0aW9uVW5pdD4KICAgICAgICAgPGV4aWY6Q29sb3JTcGFjZT42NTUzNTwvZXhpZjpDb2xvclNwYWNlPgogICAgICAgICA8ZXhpZjpQaXhlbFhEaW1lbnNpb24+NTEwPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjUxMDwvZXhpZjpQaXhlbFlEaW1lbnNpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgIAo8P3hwYWNrZXQgZW5kPSJ3Ij8+s+tqrgAAACBjSFJNAAB6JQAAgIMAAPn/AACA6QAAdTAAAOpgAAA6mAAAF2+SX8VGAACjYklEQVR42uz9d5gd13nniX8r33xv5xzRjW6ERiIBEAQDKJJgFkmRoiSKlGVZo5H183jC7szPY6/ntz+PPV7P7s7MMzvBY1vW2BYpiaRIimIOAEiCBEgARE6dcw4338q1f1wAAgkQ6HBvd1Xd9/M8eiQhNKpOnXO+533PG5jf/u3nQRCEfclEMx6GZcoBVDAsUwWgmmGYWoZhKsGggmGYcjAIMAzjByCBgcgwjAcAy7CM+LkfxoBhGIa5/Jcsy7Jgwfrcr5mWCsC0LEuGBRWAYllWChaSlmVNwsKEZVnjlmUNAxi1TGsMwIRlWpPeiFemr0YQ9oWnISCIlUGOyRGwWMUwTDvDMK0My7QwDLMKLMoZhokwLONnWEb0RrxMPp+DYRgGDD73bzAs47nwP30L/XmWZVmWaamWaaUsy4rCxKRlWT2WaXVbltVlWdY5mOjxhD1RmgUEQcJPEO4S97hczzBMB8MyGxmW2cCwTBvDMvUsx0Y8YQ/rxndmGIZhOEYCBwlAMYBmADddcUAwLdM0zKhlWoOWaZ23TOuEZVrHLcs66Ql5Bmn2EAQJP0HYWeBXMSxzA8MyO1mW3cxwTDvLsaWekIeh0fmSAwLLsBzLFV84HGwC8I3LDgWWaZjTlmGdM03zqGVaH1mmdcQT8vTQyBEECT9BLBuZaEZgOXYzwzK3MyxzK8uxm1merSGBz/mhgOFYrgwCygDcCuD3Lx0IdHPENMyjlml9aJnW+6ZhHvVGvBqNGkGQ8BNELiz5NSzH3suwzN0sz271hD0lXwyOI5b5QCBytRy4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'
      }],
      debouncer: 500,
      gitHubLink: 'https://github.com/ThunderQuoteTeam/PaynowQR',
      customLogo: false,
      generating: false,
      inMobile: window.innerWidth <= 991 ? true : false,
      rawDate: '',
      money: {
        decimal: '.',
        thousands: '',
        prefix: '',
        suffix: '',
        precision: 2,
        masked: false /* doesn't work with directive */
      },
      qrForm: {
        uen:'',           //Required: UEN of company
        amount : '500.00',               //Specify amount of money to pay.
        editable: true,             //Whether or not to allow donor to edit payment amount. Defaults to false if amount is specified
        expiry: '',         //Set an expiry date for the Paynow QR code (YYYYMMDD). If omitted, defaults to 5 years from current time.
        refNumber: '',   //Reference number for Paynow Transaction. Useful if you need to track payments for recouncilation.
        company:  ''   //Company name to embed in the QR code. Optional.               
      },
      output: null,
      qrLogo: {
        src: null,
        name: null,
        logoHeight: '67',
        logoWidth: '76',
        logoHeightWidth: '76',
        bgColor: '#FFFFFF',
        bgTransparent: false
      },
      qrBg: {
        src: null,
        name: null,
        autoBgColor: false,
        bgImgAlpha: '0.5',
      },
      qr: {
        color: {
          dark: '#90137B',
          light: '#FFFFFF',
        },
        border: {
          color: '#FFFFFF',
          size: 12
        }
      },
      qRCodeImage: null,
      qrLogoText: {
        x: 25,
        y: -23,
        width: 450,
        height: 500,
        fontSize: 98,
        lineHeight: 130,
        fontWeight: 'normal',
        text: 'Event\n2020',
        textAlign: 'center',
        textColor: '#000000',
      },
      toggleNewTop: true
    }
}


export default {
  name: 'PaynowQRDemo',
  data: getInitialData,
  components: {
    welcomeMessage
  },
  directives: {money: VMoney},
  watch: {
    inMobile(){
      this.reloadOutput();
    },
    customLogo(val){
      switch(val){
        case true:
          this.removeLogo();
          break;
        case false:
          this.useDefaultLogo();
          break;
        case 'text':
          this.renderText();
          break;
      }
    },
    rawDate(val){
      if(val){
        this.qrForm.expiry = val.replace(/-/g, '');
        return;
      }
      this.reloadOutput();
    },
    qrForm: {
      handler(){
        this.reloadOutput();
      },
      deep: true
    },
    qr: {
      handler(){
        this.reloadOutput();
      },
      deep: true
    },
    'qr.border.color': {
      handler(){
        this.reloadOutput();
      },
      deep: true
    },
    qrLogo:{
      handler(){
        this.reloadOutput();
      },
      deep: true
    },
    qrLogoText:{
      handler(){
        this.renderText();
      },
      deep: true
    },
    qrBg:{
      handler(){
        this.reloadOutput();
      },
      deep: true
    },
  },
  mounted() {
    window.addEventListener('resize', this.resizeHandler)
    window.addEventListener('scroll', this.scrollHandler);
    this.useDefaultLogo();
    this.rawDate = moment().add('years', 5).format('YYYY-MM-DD');
    this.reloadOutput();
    
  },
  destroyed(){
    window.removeEventListener('resize', this.resizeHandler);
    window.addEventListener('scroll', this.scrollHandler);
  },
  methods: {
    padZero(str,len) {
      len = len || 2;
      let zeros = new Array(len).join('0');
      return (zeros + str).slice(-len);
    },
    invertColor(hex) {
      if (hex.indexOf('#') === 0) {
          hex = hex.slice(1);
      }
      // convert 3-digit hex to 6-digits.
      if (hex.length === 3) {
          hex = hex[0] + hex[0] + hex[1] + hex[1] + hex[2] + hex[2];
      }
      if (hex.length !== 6) {
          throw new Error('Invalid HEX color.');
      }
      // invert color components
      let r = (255 - parseInt(hex.slice(0, 2), 16)).toString(16),
          g = (255 - parseInt(hex.slice(2, 4), 16)).toString(16),
          b = (255 - parseInt(hex.slice(4, 6), 16)).toString(16);
      // pad each with zeros and return
      return '#' + this.padZero(r) + this.padZero(g) + this.padZero(b);
    },
    addTQWaterMark(){
      let c = this.$refs.qrcode.firstChild
      let img = this.$refs.qrcode.lastChild

      let newCanvas = document.createElement('canvas');
      let ctx = newCanvas.getContext('2d')
      newCanvas.width = c.width;
      newCanvas.height = c.height;
      ctx.drawImage(c, 0, 0);
      
      
      ctx.font = "2px";
      // Create gradient
      let gradient = ctx.createLinearGradient(0, 0, newCanvas.width, 0);
      gradient.addColorStop("0", this.invertColor(this.qr.border.color));
      // Fill with gradient
      ctx.fillStyle = gradient;
      ctx.fillText("Generated by ThunderQuote", newCanvas.width - 130, newCanvas.height - 3);
      
      img.setAttribute('src', newCanvas.toDataURL());
      this.generating = false;
    },
    renderText(){
      if(this.customLogo !== 'text') {
        return;
      }
      let c = document.getElementById('textCanvas')
      let ctx = c.getContext('2d')
      ctx.clearRect(0, 0, c.width, c.height);

      //You can use \n to define custom line breaks
      let txt = this.qrLogoText.text

      //You can also use other methods alongside this
      ctx.fillStyle = this.qrLogoText.textColor //red color text

      //canvasTxt.font = 'Verdana'
      canvasTxt.fontSize = this.qrLogoText.fontSize;
      canvasTxt.fontWeight = this.qrLogoText.fontWeight;
      canvasTxt.align = this.qrLogoText.textAlign
      canvasTxt.lineHeight = Number(this.qrLogoText.lineHeight)
      canvasTxt.debug = false //shows debug info
      canvasTxt.drawText(ctx, txt, this.qrLogoText.x, this.qrLogoText.y, this.qrLogoText.width, this.qrLogoText.height)
      let renderedText = c.toDataURL();
      let imgLogo = document.getElementById('logoimg');
      this.$set(this.qrLogo, 'src', renderedText);
      imgLogo.setAttribute('src', renderedText);
    },
    resetGenerator(){
      Object.assign(this.$data, getInitialData());
    },
    useDefaultLogo(){
      let payNowLogo = "data:image/png;base64,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";
      let imgLogo = document.getElementById('logoimg');
      this.$set(this.qrLogo, 'src', payNowLogo);
      imgLogo.setAttribute('src', payNowLogo);
    },
    fixAmount(){
      this.qrForm.amount ? this.$set(this.qrForm, 'amount', Number(this.qrForm.amount).toFixed(2).toString()) : this.$set(this.qrForm, 'amount', '0.00');
    },
    resizeHandler(val){
      val.target.innerWidth <= 991 ? this.inMobile = true : this.inMobile = false;
    },
    scrollHandler(val){
      let scroll = val.target.body.getBoundingClientRect().top
      scroll >= -145 ? this.toggleNewTop = true : this.toggleNewTop = false;
      //val.target.innerWidth <= 991 ? this.inMobile = true : this.inMobile = false;
    },
    downloadqRCodeImage() {
      let e = this.$refs.qrcode;
      const downloadLink = document.createElement('a');
      document.body.appendChild(downloadLink);
      downloadLink.href = e.children[1].src;
      downloadLink.target = '_self';
      downloadLink.download = 'PayNow QR Code';
      downloadLink.click(); 
      document.body.removeChild(downloadLink);

      this.$bvToast.toast(`PayNow QR Code output has been downloaded`, {
          title: 'Success!',
          autoHideDelay: 2000,
          variant: 'success',
      });
    },
    removeLogo() {
      let logoimg = document.getElementById("logoimg");
      logoimg.removeAttribute("src");
      this.qrLogo.src = null;
    },
    removeBackground() {
      let bgimg = document.getElementById("bgimg");
      bgimg.removeAttribute("src");
      this.qrBg.src = null;
    },
    async reloadOutput(){
        await this.$nextTick();
        if(!this.qrForm.uen){
          return;
        }

        let obj = new PaynowQR(this.qrForm);
        this.output = obj.output();

        let options = {
          text: this.output,
          width: 256,
          height: 256,
          colorDark : this.qr.color.dark,
          colorLight : this.qr.color.light,
          correctLevel : QRCode.CorrectLevel.H,
          dotScale: 1, 
          
          quietZone: this.qr.border.size,
          quietZoneColor: this.qr.border.color,
      
          
          onRenderingStart: () => { this.generating = true },
          onRenderingEnd: () => { this.addTQWaterMark()},

      }

        let uploadedLogoImage = document.getElementById("logoimg");
        if (uploadedLogoImage && uploadedLogoImage.src) {
          options.logo =  uploadedLogoImage.src;
          //options.logoWidth =  Number(this.qrLogo.logoWidth);
          options.logoWidth = Number(this.qrLogo.logoHeightWidth);
          options.logoHeight = Number(this.qrLogo.logoHeightWidth);
          //options.logoHeight =  Number(this.qrLogo.logoHeight);
          options.logoBackgroundColor =  this.qrLogo.bgColor;
          options.logoBackgroundTransparent = this.qrLogo.bgTransparent;
        }

        let uploadedBackgroundImage = document.getElementById("bgimg");
        if (uploadedBackgroundImage && uploadedBackgroundImage.src) {
          options.backgroundImage =  uploadedBackgroundImage.src;
          options.backgroundImageAlpha = this.qrBg.bgImgAlpha;
          options.autoColor = this.qrBg.autoBgColor;
        }

        if (this.$refs && this.$refs.qrcode) {
           this.$refs.qrcode.innerHTML = '';
           new QRCode(this.$refs.qrcode, options);
           
        }

        
    },
    loadLogoImage(){
        if (!this.qrLogo.src) {
          return;
        }

        let selectedFile = this.qrLogo.src;
        let imgtag = document.getElementById("logoimg");
        let reader = new FileReader();

        reader.onload = (event) => {
          imgtag.src = event.target.result;
          this.reloadOutput();
        };

        reader.readAsDataURL(selectedFile);
        
    },
    loadBgImage(){
        if (!this.qrBg.src) {
          return;
        }

        let selectedFile = this.qrBg.src;
        let imgtag = document.getElementById("bgimg");
        let reader = new FileReader();

        reader.onload = (event) => {
          imgtag.src = event.target.result;
          this.reloadOutput();
        };

        reader.readAsDataURL(selectedFile);
    },
    copyToClipboard() {
      const el = document.createElement('textarea');
      let e = document.getElementById('textarea-code-output');
      el.value = e.value;
      document.body.appendChild(el);
      el.select();
      document.execCommand('copy');
      document.body.removeChild(el);

      this.$bvToast.toast(`The text has been successfully copied to your clipboard!`, {
          title: 'Copied!',
          autoHideDelay: 2000,
          variant: 'success',
      });
    }
  }
}
</script>

<style lang="scss">
$primary: #4e3961;

@import "~@openfonts/montserrat_all/index.css";

html  {
  scroll-behavior: smooth;
  width: 100%;
  height: 100%;
  margin: 0;
  color: #fff;
  font-family: 'Montserrat';
}

.main-container {
  width: 100%;
  height: 100%;
  margin: 0;
  font-family: 'Montserrat';
}

.color-white {
  color: #fff;
}


body {
  color: #fff !important;
  background-color: #8788d6 !important;
}

.main-container, #app {
  overflow: unset;
}

.main-box {
  -webkit-box-shadow: 0px 13px 34px -13px #000000;
  -moz-box-shadow: 0px 13px 34px -13px #000000;
  box-shadow: 0px 13px 34px -13px #000000;
  border-radius: 10px;
  max-width: 900px;
  margin-top: 0px;
}

.col-box-first {
  background: #5455bbe0;
}

.corners-first {
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}

.corners-mobile-first {
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.corners-second {
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
}

.corners-mobile-second {
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
}

.col-box-second {
  background: #898bc3;
}

.col-row, .col-box {
  height: auto;
}

.full-width {
  width: 100%;
}

.output-text {
  cursor: pointer;
  color: rgb(29, 29, 29);
  word-break: break-all;
}

.pseudo-link {
  cursor: pointer;
}

h1 {
  text-align: center;
}

#qr-image {
  cursor: pointer;
}

.qr-image-container {
  min-height: 291px;
}

.full-height {
  height: 100%;
}

.tip {
  display: block;
  width: 100%;
  height: 100%;
  padding: 0.65rem 0.75rem;
  font-size: 0.9rem;
  text-align: center;
  font-style: italic;
  font-weight: 400;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.second-box-content-container {
  position: sticky;
  top: 0;
}

.text-italic {
  font-style: italic;
}

.border-dim-white {
  border-color: rgba(255,255,255,0.6);
}

.settings-section {
  text-align: center;
}

.thunderquote-footer {
  color: #FFFFFF;
  position: fixed;
  bottom: 0;
  display: flex;
  justify-content: flex-end;
}

.github-header {
  position: fixed;
  top: 0;
  display: flex;
  justify-content: flex-end;
  right: 0;
}


.github-corner svg{
  position:absolute;
  right:0;
  top:0;
  mix-blend-mode:darken;
  color:#FFFFFF;
  fill:#000000;}

.github-corner:hover .octo-arm{
  animation:octocat-wave .56s;
}

@keyframes octocat-wave{
  0%, 100%{
    transform:rotate(0);
  }
  20%, 60%{
    transform:rotate(-20deg);
  }
  40%, 80%{
    transform:rotate(10deg);
    }
}

.no-style-link {
  color: inherit;
  text-decoration: none;
}

.no-style-link:hover {
  color: inherit;
  text-decoration: none;
}

.text-black {
  color: #000000;
}

.btn-custom-primary {
  color: #fff;
  background-color: $primary;
  border-color: $primary;
}

.bg {
  background: red;
  height: 100%;
}

.diagonal-box {
  position: fixed;
  width: 100vw;
  top: 50%;
  height: 166px;
  z-index: -1;
  outline: 20px solid transparent;
  background: #7273d8;
  overflow: hidden;
  transform: skewY(-44deg);
} 

.diagonal-box-2 {
  position: fixed;
  width: 100vw;
  top: 73%;
  height: 300px;
  z-index: -1;
  bottom: 0px;
  outline: 20px solid transparent;
  background: #5f5fbe;
  overflow: hidden;
  transform: skewY(-44deg);
} 

.new-box {
  background: #5455bbe0;
  width: 800px;
  border-radius: 4px;
  /* border: 3px solid #8c8cfa; */
  box-shadow: 4px 17px 52px -3px #6768c5;
}

.new-box-2 {
  background: #888bc3;
  width: 800px;
  border-radius: 4px;
  /* border: 3px solid #8c8cfa; */
  box-shadow: 4px 17px 52px -3px #6768c5;
}

.new-child-box {
  transition: all 0.3s ease;
  position: fixed;
  border: 0px solid #8688d6;
  background: #FFFFFF38;
  width: 100%;
  max-width: 437px;
  top: 162px;
  right: 42px;
  border-radius: 8px;
  box-shadow: 4px 17px 52px -3px #6768c5;
}

.spacer {
  height: 50px;
}

.background {
  z-index: -1;
}

.text-tq {
  color: #8788d6;
}

.header-logo {
  height: 60px;
  margin-top: 15px;
  margin-bottom: 20px;
  margin-left: 10px;
  margin-right: 10px;  
}

.logos-box {
  border-radius: 5px;
  width: 100%;
  display: flex;
  margin-top: 10px;
  margin-bottom: 10px;
  justify-content: center;
}

.white-bg {
  background: #ffffff;
}

h3 > div {
  font-size: 25px;
}

.new-top {
  top: 61px;
}
</style>

