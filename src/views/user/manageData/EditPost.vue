<template>
    <v-dialog
    v-model="dialog"
    persistent 
    width="600px">
        <template v-slot:activator="{on}">
            <v-btn
            text
            v-on="on"
            color="orange">
            Edit
            </v-btn>
        </template>
        <v-card>
            <v-card-title>
                Edit Item
            </v-card-title>
            <v-card-text>
                <v-form
                v-model="valid" 
                lazy-validation 
                ref="form">
                    <v-text-field
                    outlined
                    v-model="name"
                    label="Name"
                    ></v-text-field>
                    <v-textarea
                    outlined
                    v-model="description"
                    name="input-7-1"
                    label="Description"
                    ></v-textarea>
                    <v-file-input
                    prepend-icon="mdi-camera"
                    accept="image/*"
                    label="Select Image"
                    v-model="file"
                    show-size
                    ></v-file-input>
                    <p>Current Image: <a v-if="oldImage" :href="oldImage" target="_blank">Here</a></p>
                    <v-card-actions class="justify-center">
                        <v-btn
                        dark
                        elevation="2"
                        color="orange"
                        @click="updateItem"
                        :loading="isLoading">
                        Update
                        </v-btn>
                        <v-btn
                        dark
                        color="red"
                        @click="dialog = !dialog">
                        Close
                        </v-btn>
                    </v-card-actions>
                </v-form>
            </v-card-text>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from 'vuex'
import {auth, storage, itemsCollection} from '../../../firebase'
export default {
    name: 'EditItem',
    components: {

    },
    props: ['item', 'index'],
    data() {
        return {
            name: '',
            description: '',
            price: '',
            file: null,
            oldImage: '',
            dialog: false,
            valid: false,
            isLoading: false,
        }
    },
    computed: {
        ...mapActions(['getItemsByUser'])
    },
    methods: {
        async updateItem(){
            try {
                this.isLoading = true
                let data = {
                    userId: auth.currentUser.uid,
                    name: this.name,
                    description: this.description,
                    price: this.price,
                }

                if (this.file) {
                    // delete old image
                    const fileRefOld = this.item.img
                    await storage.ref(fileRefOld).delete()

                    //upload new image
                    const fileRef = 'uploads/items/' + this.file.name
                    await storage.ref(fileRef).put(this.file)
                    data.image = fileRef
                } else {
                    data.image = this.item.img
                }
                await itemsCollection.doc(this.item.id).update(data)
                this.isLoading = false
                this.dialog = false
                await this.getItemsByUser
                alert('item updated')
            }catch(error){
                console.log(error);
            }  
        },
        setData(){
            if(this.item){
                this.name = this.item.name
                this.description = this.item.description
                this.oldImage = this.item.image
            }
        },
    },
    mounted(){
        this.setData()
    }
}
</script>

<style>

</style>