<template>

  <FabEnigme v-if="indices.length" :enigme="enigme" :indices="indices" />

  <IntituleEtEnonceEnigme :enigme="enigme" />

  <template v-if="!loading">
    <template v-if="enigme?.solutionUniques.length">
      <ReponseSolutionUnique
        :enigmeId="enigmeId"
        @incorrect="answerIsNotCorrect"
        @correct="answerIsCorrect"
      />
    </template>

    <template v-else-if="enigme?.solutionAChoixes.length">
      <ReponseSolutionAChoixes
        :enigmeId="enigmeId"
        :solutions="enigme.solutionAChoixes"
        @incorrect="answerIsNotCorrect"
        @correct="answerIsCorrect"
      />
    </template>

    <template v-else-if="enigme?.solutionMultiples.length">
      <ReponseSolutionMultiples
        :enigmeId="enigmeId"
        :solutions="enigme.solutionMultiples"
        @incorrect="answerIsNotCorrect"
        @correct="answerIsCorrect"
      />
    </template>

    <DialogEnigme
      :dialog="dialog"
      :title="title"
      :message="message"
      :image="imageCorrectResponse"
      :redirection="redirection"
      @close="close"
    />
  </template>
</template>

<script>
import IntituleEtEnonceEnigme from "@/components/IntituleEtEnonceEnigme.vue";
import ReponseSolutionUnique from "@/components/ReponsesEnigmes/ReponseSolutionUnique.vue";
import FabEnigme from "../components/FabEnigme.vue";
import DialogEnigme from "@/components/DialogEnigme.vue";
import ReponseSolutionAChoixes from "@/components/ReponsesEnigmes/ReponseSolutionAChoixes.vue";
import ReponseSolutionMultiples from "@/components/ReponsesEnigmes/ReponseSolutionMultiples.vue";
import { mapGetters } from "vuex";

export default {
  components: {
    IntituleEtEnonceEnigme,
    ReponseSolutionUnique,
    FabEnigme,
    DialogEnigme,
    ReponseSolutionAChoixes,
    ReponseSolutionMultiples,
  },
  data: () => ({
    loading: true,
    dialog: false,
    enigmeId: null,
    enigme: null,
    indices: [],
    title: "",
    message: "",
    imageCorrectResponse: null,
    redirection: null,
  }),
  async created() {
    this.enigmeId = this.$route.params.enigmeId;
    if (this.enigmeId === undefined) {
      this.$router.push("difficultes");
    } else {
      const response = await this.$axios.get("enigmes/" + this.enigmeId);
      this.enigme = response.data;

      this.indices = [
        this.enigme.indice1,
        this.enigme.indice2,
        this.enigme.indice3,
      ];

      this.loading = false;
    }
  },
  computed: {
    ...mapGetters("userStore", ["user"]),
  },
  methods: {
    answerIsNotCorrect(message_response_is_incorrect) {
      this.title = "Réponse fausse... !";
      this.message = message_response_is_incorrect;
      this.imageCorrectResponse = null;
      this.redirection = null;
      this.dialog = true;
    },
    async answerIsCorrect(message_response_is_correct, image_response_is_correct) {
      this.title = "Bien joué !";
      this.message = message_response_is_correct;
      this.imageCorrectResponse = image_response_is_correct;
      this.redirection = {
        name: "enigmes",
        params: {
          difficulty: this.enigme.difficulty["@id"],
          labelDifficulty: this.enigme.difficulty.difficulty,
        },
      };
      this.dialog = true;
      const enigmeResolue = await this.$axios.get('enigme_resolues?and[enigme]='+this.enigme['@id']+'&and[user]='+this.user['@id'])
      if (enigmeResolue.data['hydra:totalItems'] === 0) {
        await this.$axios.post('enigme_resolues', {
          'enigme': this.enigme['@id'],
          'user': this.user['@id'],
        })
      }
    },
    close() {
      this.dialog = false;
    },
  },
};
</script>