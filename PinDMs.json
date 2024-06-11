import { Plugin, registerPlugin } from 'enmity/managers/plugins';
import { getByName, getByProps } from 'enmity/metro';
import { getIDByName } from "enmity/api/assets";
import { create } from 'enmity/patcher';
import findInReactTree from 'enmity/utilities/findInReactTree';
import Manifest from './manifest.json';
import { FormRow } from 'enmity/components';
import { React, Toasts } from 'enmity/metro/common';
import { get, set, subscribe, unsubscribe } from 'enmity/api/settings';

const Patcher = create("pin-dms")

const ConnectedPrivateChannelsModule = getByName("ConnectedPrivateChannels", { default: false })
const LazyActionSheet = getByProps("openLazy", "hideActionSheet")
const ChannelLongPressActionSheetModule = getByName("ChannelLongPressActionSheet", { default: false })
const Icon = getByName("Icon")

const Pin = getIDByName("ic_message_pin")

const PinDMs: Plugin = {
   ...Manifest,

   onStart() {
      Patcher.after(ConnectedPrivateChannelsModule, "default", (self, args, res) => {
         const forceUpdate = React.useState({})[1];

         // refresh on pins change
         React.useEffect(() => {
            function onPinsChange() {
               forceUpdate({});
             }
       
             subscribe(Manifest.name, onPinsChange);
             return () => unsubscribe(Manifest.name, onPinsChange);
         }, [])

         // Get pins
         const pins = get(Manifest.name, "pins", []) as string[]

         pins.forEach((pinnedDmId) => {
            // Find dm's index in array
            const dmIndex = res.props.privateChannelIds.findIndex((id) => id === pinnedDmId)

            // if found
            if (dmIndex !== -1) {
               // remove from existing location
               res.props.privateChannelIds.splice(dmIndex, 1)
               // move to start of array
               res.props.privateChannelIds.unshift(pinnedDmId)
            }
         })
      })

      Patcher.after(ChannelLongPressActionSheetModule, "default", (_, args, res) => {
         const channel = res.props.channel

         if (channel.guild_id === null) {
            Patcher.after(res, "type", (self, args, connectedRes) => {
               // find dm section in action sheet
               const dmSection = findInReactTree(connectedRes, (node) => node.key === "dm")

               // Get list of pins
               const pins = get(Manifest.name, "pins", []) as string[]

               // Helpful ternary operators
               const pinned = pins.includes(channel.id)
               const dmType = channel.rawRecipients.length == 1 ? "DM" : "Group" 

               const pinCallback = () => {  
                  if (pinned) {
                     set(Manifest.name, "pins", pins.filter((pin) => pin !== channel.id))
                  } else {
                     set(Manifest.name, "pins", [...pins, channel.id])
                  }

                  // Notify user the DM/Group was pinned
                  Toasts.open({content: !pinned ? `Pinned ${dmType}` : `Unpinned ${dmType}`, source: Pin })
                  // Hide the action sheet
                  LazyActionSheet.hideActionSheet()
               }
               
               dmSection.props.children.push(<FormRow leading={<Icon source={Pin} />} label={!pinned ? `Pin ${dmType}` : `Unpin ${dmType}`} onPress={pinCallback} />)
            })
         }
      })
   },

   onStop() {
      Patcher.unpatchAll()
   },
};

registerPlugin(PinDMs);
