<template><div class="hero">
  <div class="container mt-3">
    <div class="row">
      <!-- Short info card -->
      <div class="col-md-4">
        <div class="card">
          <div class="card-body" v-if="meta">
            <div class="d-flex flex-column align-items-center text-center">
              <img
                v-if="meta && meta.thumbnailUrl"
                :src=meta.thumbnailUrl
                alt="Profile thumbnail"
                class="rounded-circle"
                width="150"
              />
              <h3 class="inside" v-if="meta && !meta.thumbnailUrl && editMode">
                Please choose a profile picture
              </h3>
              <input class="inside"
                v-if="editMode"
                type="file"
                @change="thumbnailChanged"
                ref="thumbnailFileInput"
              />
              <div class="mt-3">
                <h4 v-if="!editMode">{{ meta.displayName }}</h4>
                <input
                  type="text"
                  v-model="meta.displayName"
                  v-if="editMode"
                  placeholder="Display name"
                />
                <p class="text-secondary mb-1" v-if="meta && meta.emails">
                  <span
                    v-for="(email, idx) in meta.emails"
                    :key="'email_' + idx"
                  >
                    <a
                      :href="'mailto:' + email.value"
                      class="link-secondary"
                      v-if="email.primary"
                      >{{ email.value }}</a
                    >
                  </span>
                </p>
                <p class="text-secondary mb-1">
                  {{ address }}
                </p>
                <button class="btn btn-outline-dark" @click="copyToClipboard">
                  <i class="bi bi-clipboard"></i>   
                </button>
                <!-- <button class="btn btn-outline-dark" @click="showQRPanel">
                  <i class="bi bi-qr-code"></i>
                </button> -->
              </div>
            </div>
          </div>
        </div>
        <div class="d-flex flex-column align-items-center text-center mt-3">
          <button
            class="btn btn-lg btn-outline-dark w-100"
            v-if="!editMode"
            @click="jumpToEditMetadata"
          >
            Create/Edit your confyg data
          </button>
          <button
            class="btn btn-lg btn-outline-primary w-100"
            v-if="editMode"
            @click="connect"
          >
            MINT NFT
          </button>
          <button
            class="btn btn-lg btn-outline-dark mt-3 w-100"
            v-if="editMode"
            @click="generateJSON"
          >
            Create your Confyg Metadata
          </button>
          <button class="btn btn-lg btn-outline-dark mt-3 w-100" id="gd">Getdata</button>
        </div>
      </div>

      <!-- Detailed info card -->
      <div class="col-md-8">
        <div class="card">
          <div class="card-body">
            <!-- Basic data -->
            <div class="row">
              <h5 class="inside">Basic data</h5>
              <hr class="col-10" />
            </div>
            <div class="row" v-if="meta">
              <div class="col-3">
                <h6 class="inside">Full Name</h6>
              </div>
              <div class="col-9 text-secondary" v-if="!editMode && meta.name">
                {{ meta.name.formatted }}
              </div>
              <div class="col-9 text-secondary" v-if="editMode">
                <input
                  type="text"
                  placeholder="First Name"
                  v-model="givenName"
                />
                <input
                  type="text"
                  placeholder="Last Name"
                  v-model="familyName"
                />
              </div>
            </div>
            <div class="row mt-4">
              <h5 class="inside">About me</h5>
              <hr class="col-10" />
            </div>
            <div class="row" v-if="meta">
              <div
                class="col-12 text-secondary"
                v-if="!editMode && meta.aboutMe"
              >
                {{ meta.aboutMe }}
              </div>
              <div class="col-12 text-secondary" v-if="editMode">
                <textarea
                  v-model="meta.aboutMe"
                  placeholder="Some words about you..."
                  class="w-75"
                ></textarea>
              </div>
            </div>
            <!-- E-mail addresses -->
            <div class="row mt-4">
              <h5 class="inside">E-mail addresses</h5>
              <hr class="col-10" />
            </div>
            <div
              class="row mt-1"
              v-for="(email, idx) in meta.emails"
              :key="'email_' + idx"
            >
              <div class="col-12">
                <button
                  class="btn btn-danger btn-sm me-1"
                  v-if="editMode"
                  @click="deleteEmail(idx)"
                >
                  x
                </button>
                <input
                  class="form-check-input"
                  type="checkbox"
                  name="primaryEmail"
                  :value="idx"
                  :checked="email.primary"
                  @input="setPrimaryEmail(idx)"
                  v-if="editMode"
                />
                <a
                  :href="'mailto:' + email.value"
                  class="link-secondary"
                  target="_blank"
                >
                  {{ email.value }}
                </a>
              </div>
            </div>
            <div class="row mt-1" v-if="editMode">
              <div class="col-12">
                <button class="btn btn-primary btn-sm me-1" @click="addEmail">
                  +</button
                ><input
                  type="text"
                  placeholder="E-mail address"
                  class="w-75"
                  v-model="editModelEmail"
                />
              </div>
            </div>
            <!-- IM contacts -->
            <div class="row mt-4">
              <h5 class="inside">Your Contacts</h5>
              <hr class="col-10" />
            </div>
            <div class="row" v-for="(im, idx) in meta.ims" :key="'im_' + idx">
              <div class="col-3">
                <h6>
                  <button
                    class="btn btn-danger btn-sm me-1"
                    v-if="editMode"
                    @click="deleteIM(idx)"
                  >
                    x</button
                  >{{
                    im.type.substring(0, 1).toUpperCase() + im.type.substring(1)
                  }}
                </h6>
              </div>
              <div class="col-9 text-secondary">
                <a :href="getIMUrl(im.type, im.value)" class="link-secondary">{{
                  im.value
                }}</a>
              </div>
            </div>
            <div class="row" v-if="editMode">
              <div class="col-3">
                <button class="btn btn-primary btn-sm me-1" @click="addIM">
                  +</button
                ><input
                  type="text"
                  placeholder="Contacts"
                  class="w-75"
                  v-model="editModelIM.type"
                />
              </div>
              <div class="col-9 text-secondary">
                <input
                  type="text"
                  placeholder="Username"
                  class="w-75"
                  v-model="editModelIM.username"
                />
              </div>
            </div>
            <!-- Accounts -->
            <div class="row mt-4">
              <h5 class="inside">Accounts</h5>
              <hr class="col-10" />
            </div>
            <div
              class="row"
              v-for="(account, idx) in meta.accounts"
              :key="'acc_' + idx"
            >
              <div class="col-3">
                <h6>
                  <button
                    class="btn btn-danger btn-sm me-1"
                    v-if="editMode"
                    @click="deleteAccount(idx)"
                  >
                    x
                  </button>
                  <img
                    :src="
                      'https://www.google.com/s2/favicons?sz=18&domain_url=' +
                      account.domain
                    "
                    v-if="!editMode"
                  />
                  {{ account.domain }}
                </h6>
              </div>
              <div class="col-9 text-secondary">
                <a
                  :href="getAccountUrl(account.domain, account.username)"
                  class="link-secondary"
                  target="_blank"
                  >{{ account.username }}</a
                >
              </div>
            </div>
            <div class="row" v-if="editMode">
              <div class="col-3">
                <button class="btn btn-primary btn-sm me-1" @click="addAccount">
                  +</button
                ><input
                  type="text"
                  placeholder="Domain"
                  class="w-75"
                  v-model="editModelAccount.domain"
                />
              </div>
              <div class="col-9 text-secondary">
                <input
                  type="text"
                  placeholder="Username"
                  class="w-75"
                  v-model="editModelAccount.username"
                />
              </div>
            </div>
            <!-- Webpages -->
            <div class="row mt-4">
              <h5 class="inside">Web</h5>
              <hr class="col-10" />
            </div>
            <div v-if="meta && meta.urls">
              <div class="row" v-for="(webpage, idx) in meta.urls" :key="idx">
                <div class="col-12">
                  <h6>
                    <button
                      class="btn btn-danger btn-sm me-1"
                      v-if="editMode"
                      @click="deleteWebpage(idx)"
                    > 
                    </button>
                    <img
                      :src="
                        'https://www.google.com/s2/favicons?sz=18&domain_url=' +
                        webpage.value
                      "
                      v-if="!editMode"
                    />
                    <a
                      :href="webpage.value"
                      class="link-secondary"
                      target="_blank"
                    >
                      {{ webpage.value }}
                    </a>
                  </h6>
                </div>
              </div>
            </div>
            <div class="row" v-if="editMode">
              <div class="col-12">
                <button class="btn btn-primary btn-sm me-1" @click="addWebpage">
                  +</button
                ><input
                  type="text"
                  placeholder="Webpage URL"
                  class="w-75"
                  v-model="editModelWebpage"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- JSON popup -->
    <div class="modal" tabindex="-1" ref="json_popup">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Generated JSON metadata</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div
            class="modal-body d-flex flex-column align-items-center"
            style="max-height: 500px"
          >
            <pre class="w-100" v-html="generatedJSON"></pre>
          </div>
        </div>
      </div>
    </div>
</div>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import MyEthMetaClient from "myethmeta";
import { EthereumAddressMetadataJSONSchema } from "myethmeta";
import QRCode from "qrcode";
import { Modal } from "bootstrap";
import copy from "copy-to-clipboard";
import detectEthereumProvider from "@metamask/detect-provider";
import Web3 from "web3";
import { AbiItem } from "web3-utils";
import myethmeta_abi from "./myethmeta_abi.json";
import {Contract, ethers, Result} from "ethers";
import { BrowserProvider, parseUnits } from "ethers";
 import axios from "axios";
 import { Buffer } from "buffer";
 import pinataSDK, { PinataPinResponse } from "@pinata/sdk";
@Component
class App extends Vue {
  public address: string = null;
  public meta: EthereumAddressMetadataJSONSchema = {};
  public qrcodeImage: string = null;
  public editMode: boolean = false;
  public editModelIM: { type: string; username: string } = {
    type: "",
    username: "",
  };
  public editModelAccount: { domain: string; username: string } = {
    domain: "",
    username: "",
  };
  public editModelWebpage: string = "";
  public editModelEmail: string = "";
  public generatedJSON: string = "";
  public pinataKey: string = "";
  private metaClient: MyEthMetaClient = new MyEthMetaClient();

  public async mounted() {
    window.onhashchange = () => {
      this.hashChanged();
    };
    this.hashChanged();
  }

  private async hashChanged() {
    if (!window.location.hash) return;
    let address = window.location.hash.substring(1);
    if (address.length < 42) return;
    if (!address.startsWith("0x")) return;

    if (address.endsWith("/edit")) {
      this.editMode = true;
      address = address.substring(0, address.length - 5);
      const account = await this.getEthereumAccount();
      if (address != account) {
        window.location.hash = account;
        return;
      }
    } else {
      this.editMode = false;
    }
this.address = address;
this.qrcodeImage = await QRCode.toDataURL(address, { width: 400 });

    this.meta = await this.metaClient.getMetaData(address);
  }

  get familyName(): string {
    if (!this.meta || !this.meta.name || !this.meta.name.familyName) return "";
    return this.meta.name.familyName;
  }

  set familyName(value: string) {
    if (!this.meta) return;
    if (!this.meta.name)
      this.$set(this.meta, "name", {
        familyName: "",
        givenName: "",
        formatted: "",
      });
    this.meta.name.familyName = value;
    this.meta.name.formatted =
      this.meta.name.givenName + " " + this.meta.name.familyName;
  }

  get givenName(): string {
    if (!this.meta || !this.meta.name || !this.meta.name.givenName) return "";
    return this.meta.name.givenName;
  }

  set givenName(value: string) {
    if (!this.meta) return;
    if (!this.meta.name)
      this.$set(this.meta, "name", {
        familyName: "",
        givenName: "",
        formatted: "",
      });
    this.meta.name.givenName = value;
    this.meta.name.formatted =
      this.meta.name.givenName + " " + this.meta.name.familyName;
  }

  public getIMUrl(type: string, value: string) {
    let url = "#";
    if (type == "skype") url = "skype:" + value;
    if (type == "telegram") url = "https://telegram.me/" + value;
    return url;
  }

  public getAccountUrl(domain: string, username: string) {
    let url = "#";
    if (domain == "twitter.com") url = "https://twitter.com/@" + username;
    if (domain == "github.com") url = "https://github.com/" + username;
    return url;
  }

  public getGatewayUrl(uri: string) {
    return this.metaClient.getGatewayURL(uri);
  }

 

  public deleteIM(idx: number) {
    this.meta.ims.splice(idx, 1);
  }

  public addIM() {
    if (!this.meta) return;
    if (!this.meta.ims) this.$set(this.meta, "ims", []);
    this.meta.ims.push({
      type: this.editModelIM.type,
      value: this.editModelIM.username,
    });
    this.editModelIM = {
      type: "",
      username: "",
    };
  }

  public deleteAccount(idx: number) {
    this.meta.accounts.splice(idx, 1);
  }

  public addAccount() {
    if (!this.meta) return;
    if (!this.meta.accounts) this.$set(this.meta, "accounts", []);
    this.meta.accounts.push({
      domain: this.editModelAccount.domain,
      username: this.editModelAccount.username,
    });
    this.editModelAccount = {
      domain: "",
      username: "",
    };
  }

  public deleteWebpage(idx: number) {
    this.meta.urls.splice(idx, 1);
  }

  public addWebpage() {
    if (!this.meta) return;
    if (!this.meta.urls) this.$set(this.meta, "urls", []);
    this.meta.urls.push({ value: this.editModelWebpage });
    this.editModelWebpage = "";
  }

  public deleteEmail(idx: number) {
    this.meta.emails.splice(idx, 1);
  }

  public addEmail() {
    if (!this.meta) return;
    if (!this.meta.emails) this.$set(this.meta, "emails", []);
    this.meta.emails.push({ value: this.editModelEmail, primary: false });
    this.editModelEmail = "";
  }

  public setPrimaryEmail(idx: number) {
    for (let email of this.meta.emails) email.primary = false;
    this.meta.emails[idx].primary = true;
  }

  public copyToClipboard() {
    copy(this.address);
  }

  private async getEthereumAccount(): Promise<string> {
    const ethereum: any = await detectEthereumProvider();
    if (ethereum) {
      const accounts = await ethereum.request({
        method: "eth_requestAccounts",
      });
      const account = accounts[0];
      return account;
    } else {
      alert("Please install MetaMask or use a web3 browser!");
    }
    return "";
  }

  public async jumpToEditMetadata() {
    const account = await this.getEthereumAccount();
    window.location.hash = account + "/edit";
  }

  public thumbnailChanged() {
    let reader = new FileReader();
    reader.onload = (e) => {
      this.$set(this.meta, "thumbnailUrl", e.target.result);
    };
    reader.readAsDataURL((this.$refs.thumbnailFileInput as any).files[0]);
  }

  public generateJSON() {
    // JSON syntax highlihter from https://stackoverflow.com/questions/4810841/pretty-print-json-using-javascript
    const syntaxHighlight = (json) => {
      json = json
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;");
      return json.replace(
        /("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g,
        function (match) {
          var cls = "number";
          if (/^"/.test(match)) {
            if (/:$/.test(match)) {
              cls = "key";
            } else {
              cls = "string";
            }
          } else if (/true|false/.test(match)) {
            cls = "boolean";
          } else if (/null/.test(match)) {
            cls = "null";
          }
          return '<span class="' + cls + '">' + match + "</span>";
        }
      );
    };

    this.generatedJSON = syntaxHighlight(
      JSON.stringify(this.meta, undefined, 2)
    );
    new Modal(this.$refs.json_popup).show();
  }
public element=document.getElementById("getdata");
  
public async connect(){
   
   const contractAddress ="0x8a9e51ffC478c6d1D4cA8049087F5BEB642280F4";
    const contractABI = [
	{
		"inputs": [],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "approved",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "Approval",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "operator",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "approved",
				"type": "bool"
			}
		],
		"name": "ApprovalForAll",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "previousOwner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "OwnershipTransferred",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "from",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "Transfer",
		"type": "event"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "approve",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			}
		],
		"name": "balanceOf",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "burn",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "getApproved",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "operator",
				"type": "address"
			}
		],
		"name": "isApprovedForAll",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "name",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "ownerOf",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "renounceOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"internalType": "string",
				"name": "uri",
				"type": "string"
			}
		],
		"name": "safeMint",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "from",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "safeTransferFrom",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "from",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			},
			{
				"internalType": "bytes",
				"name": "data",
				"type": "bytes"
			}
		],
		"name": "safeTransferFrom",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "operator",
				"type": "address"
			},
			{
				"internalType": "bool",
				"name": "approved",
				"type": "bool"
			}
		],
		"name": "setApprovalForAll",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "bytes4",
				"name": "interfaceId",
				"type": "bytes4"
			}
		],
		"name": "supportsInterface",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "symbol",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "index",
				"type": "uint256"
			}
		],
		"name": "tokenByIndex",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "index",
				"type": "uint256"
			}
		],
		"name": "tokenOfOwnerByIndex",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "tokenURI",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "totalSupply",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "from",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "tokenId",
				"type": "uint256"
			}
		],
		"name": "transferFrom",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "transferOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	}
];
    const provider = new ethers.BrowserProvider(window.ethereum);
    await provider.send("eth_requestAccounts", []);
    const signer = await provider.getSigner();
    const accounts=await provider.listAccounts();
    const account=accounts[0];
const contract = new Contract(contractAddress,contractABI,signer) 

const config = {
  method: 'get',
  url: 'https://api.pinata.cloud/data/testAuthentication',
  headers: { 
    pinata_api_key: 'fbb2b7d25cb01964e859',
pinata_secret_api_key:'a54c6d404bc12754ae8fcffea0233edb6c95dd8a984dbb379859c1fdf148ae55'
  }
};
const res = await axios(config)
console.log(res.data);
const pinata = new pinataSDK('fbb2b7d25cb01964e859', 'a54c6d404bc12754ae8fcffea0233edb6c95dd8a984dbb379859c1fdf148ae55');
const exampleJSON = this.meta;
  
pinata.pinJSONToIPFS(exampleJSON).then((result) => {
  console.log(result);
  const cid=result.IpfsHash;
  const data=result
  console.log(cid)
  contract.safeMint(account,`https://gateway.pinata.cloud/ipfs/${cid}`);
});
const getDataButton = document.getElementById("gd");
if (getDataButton !== null) {
  getDataButton.addEventListener("click", async() => {
    const totalSupp = await contract.totalSupply();
    const latestTokenId = BigInt(totalSupp)-BigInt(1);
    const uri=await contract.tokenURI(latestTokenId);
console.log(uri);
const data = {
  uri: uri
};
// Make a POST request to the API endpoint
fetch('http://localhost:3000/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  
  },
  body: JSON.stringify(data)
})
  .then(response => response.json())
  .then(responseData => {
    // Process the response data from the API
    console.log(responseData);
  })
  .catch(error => {
    console.log('Error:', error);
  });

  });
}
}

  public async publishMetadata() {
    let result = null;
    try {
      const response = await fetch(
        "https://api.pinata.cloud/pinning/pinJSONToIPFS",
        {
          method: "POST",
          headers: {
            Authorization: `Bearer ${this.pinataKey}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify(this.meta),
        }
      );
      result = await response.json();
    } catch (e) {}

    if (!result || !result.IpfsHash) {
      alert("Cannot publish data to IPFS! Please check your JWT key!");
      return;
    }

    const ethereum: any = await detectEthereumProvider();
    if (!ethereum) {
      alert("Please install MetaMask, or use a Web3 browser!");
      return;
    }

    try {
      const account = await this.getEthereumAccount();
      const web3 = new Web3(ethereum);
      const contract = new web3.eth.Contract(
        myethmeta_abi as AbiItem[],
        CONTRACT_ADDRESS
      );
      const contract_call_result = await contract.methods
        .setMetaURI("ipfs://" + result.IpfsHash)
        .send({ from: account });
    } catch (e) {
      alert("Cannot publish metadata on blockchain!");
      return;
    }
    

    new Modal(this.$refs.upload_popup).hide();
  }
}

export default App;
</script>

<style>
pre {
  outline: 1px solid #ccc;
  padding: 5px;
  margin: 5px;
}
.string {
  color: green;
 
}
.number {
  color: darkorange;
}
.boolean {
  color: blue;
}
.null {
  color: magenta;
}
.key {
  color: red;
}

.card {
  background-color: #0d0d0d;
  margin-top: 2px;
  border: 2px solid;
  margin-bottom:15px;
 
}

.hero {
  
  background: linear-gradient(90deg, rgba(0,0,0,1) 51%, rgba(26,26,26,1) 72%);
  height: 100%;
  padding: 35px;
}

.inside {
  color: white;
  
}

.btn {
  color: #000000;
  background-color: #2dd22d;
  border-color: white;
}

.btn:hover {
  background-color: #0d0d0d;
  border-color: white;
  color: #2dd22d;
  box-shadow: 2px 2px #46d246;
}
input[type="text"],textarea {
  border-radius: 8px;
  border-color: #000000;
  padding: 3px;
  margin-left: 3px;
}
</style>
