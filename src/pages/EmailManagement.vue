<template>
  <q-page padding>
    <div class="q-mt-md row">

      <!-- Available Recipients Section -->
      <div class="col-6 eml-mng-div">
        <q-card flat bordered class="eml-card">
          <q-card-section>
            <div class="text-h6">Available Recipients</div>
          </q-card-section>
          <!-- Form to add/validate a new email or search through available recepients  -->
          <q-form
            @submit.prevent="addRecipient(search)"
            ref="emailForm"
            class="q-mx-md q-pb-md"
          >
            <q-input
              v-model="search"
              outlined
              label="Search by company or email"
              hint="Press Enter to add a custom email"
              dense
              :rules="[validateEmail]"
              lazy-rules="ondemand"
            >
              <!-- Appending button for add email -->
              <template v-slot:append>
                <q-btn flat round icon="add" @click="addRecipient(search)" />
              </template>
            </q-input>
          </q-form>
          <q-list>
            <!-- Looping through grouped available recipients and creating expandable items -->
            <q-expansion-item
              v-for="group in filteredAvailableRecipients"
              :key="group.domain"
              expand-separator
              class="domain-exp"
              :label="'Domain: '+group.domain"
            >
              <!-- Headers with domain and Select All icon -->
              <template v-slot:header>
                <div class="row items-center justify-between full-width">
                  <span>Domain: {{ group.domain }}</span>
                  <q-icon 
                    size="sm" 
                    name="add_circle" 
                    color="primary" 
                    @click.stop="toggleSelectionByDomain(group.domain, true)" />
                </div>
              </template>

              <!-- Looping through emails in each group -->
              <q-item
                v-for="email in group.emails"
                :key="email"
                clickable
                @click="toggleSelection(email)"
              >
                <q-item-section>{{ email }}</q-item-section>
                <q-item-section side>
                  <q-icon name="add_circle" color="primary" />
                </q-item-section>
              </q-item>
            </q-expansion-item>
          </q-list>
        </q-card>
      </div>

      <!-- Selected Recipients Section -->
      <div class="col-6 eml-mng-div">
        <q-card flat bordered class="eml-card">
          <q-card-section>
            <div class="text-h6">Selected Recipients</div>
          </q-card-section>
          <q-list>
            <!-- Looping through grouped selected recipients and creating expandable items -->
            <q-expansion-item
              v-for="group in groupedSelectedRecipients"
              :key="group.domain"
              expand-separator
              class="domain-exp"
              :label="'Domain: '+group.domain"
            >
              <!-- Headers with domain and Deselect All icon -->
              <template v-slot:header>
                <div class="row items-center justify-between full-width">
                  <span>Domain: {{ group.domain }}</span>
                  <q-icon 
                    size="sm" 
                    name="remove_circle" 
                    color="negative" 
                    @click.stop="toggleSelectionByDomain(group.domain, false)" />
                </div>
              </template>

              <!-- Loop through emails in each group -->
              <q-item
                v-for="email in group.emails"
                :key="email"
                clickable
                @click="toggleSelection(email)"
              >
                <q-item-section>{{ email }}</q-item-section>
                <q-item-section side>
                  <q-icon name="remove_circle" color="negative" />
                </q-item-section>
              </q-item>
            </q-expansion-item>
          </q-list>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
export default {
  name: "EmailManager",

  data() {
    return {
      search: "",
      recipients: [
        { email: "ann@timescale.com", isSelected: false },
        { email: "bob@timescale.com", isSelected: false },
        { email: "brian@qwerty.com", isSelected: true },
        { email: "james@qwerty.com", isSelected: false },
        { email: "jane@awesome.com", isSelected: false },
        { email: "kate@qwerty.com", isSelected: true },
        { email: "mike@hello.com", isSelected: true },
      ],
    };
  },

  computed: {
    // Getting available recipients which are not selected
    availableRecipients() {
      return this.recipients.filter((r) => !r.isSelected);
    },

    // Getting selected recipients which are selected
    selectedRecipients() {
      return this.recipients.filter((r) => r.isSelected);
    },

    // Grouping available recipients by their domain
    groupedAvailableRecipients() {
      const groups = {};
      this.availableRecipients.forEach((recipient) => {
        const domain = recipient.email.split("@")[1];
        if (!groups[domain]) groups[domain] = [];
        groups[domain].push(recipient.email);
      });
      return Object.entries(groups).map(([domain, emails]) => ({ domain, emails }));
    },

    // Grouping selected recipients by their domain
    groupedSelectedRecipients() {
      const groups = {};
      this.selectedRecipients.forEach((recipient) => {
        const domain = recipient.email.split("@")[1];
        if (!groups[domain]) groups[domain] = [];
        groups[domain].push(recipient.email);
      });
      return Object.entries(groups).map(([domain, emails]) => ({ domain, emails }));
    },

    // Filtering available recipients based on search input
    filteredAvailableRecipients() {
      const trimmedSearch = this.search.trim().toLowerCase();
      return this.groupedAvailableRecipients.filter(
        (group) =>
          group.domain.includes(trimmedSearch) ||
          group.emails.some((email) => email.toLowerCase().includes(trimmedSearch))
      );
    },
  },

  methods: {
    /**
     * @param email an email string to compare with the other arry
     * Here we are finding an email to select/deselect and toggle from the available recipients list.
     */
    toggleSelection(email) {
      const recipient = this.recipients.find((r) => r.email === email);
      if (recipient) recipient.isSelected = !recipient.isSelected;
    },
    
    /**
     * @param {string} domain - The domain to match against recipients.
     * @param {boolean} isSelected - True to select all, false to deselect all.
     * Selects or deselects all recipients in a given domain based on the isSelected flag.
     */
    toggleSelectionByDomain(domain, isSelected) {
      this.recipients
        .filter((r) => r.email.split("@")[1] === domain && r.isSelected !== isSelected)
        .forEach((r) => (r.isSelected = isSelected));
    },

    /**
     * @param email email to be validated and added in the array. 
     * Here we are validating the email field
     * if validated then adding email to the list and resetting search value to blank
     */
    addRecipient(email) {
      this.$refs.emailForm.validate().then(value => {
        if(value) {
          this.recipients.push({ email, isSelected: false });
          this.search = "";
        }
      })
    },
    
    /**
     * @param value email to be validated
     * Here we are first checking if user has added any email and 
     * if added then we are validating the email using regex
     */
    validateEmail(value) {
      if (!value.length) return "Please enter email";
      const emailPattern = /^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/;
      return emailPattern.test(value) || "Invalid email format."; // Return error message if invalid
    }
  },
};
</script>

<style>
.q-page {
  max-width: 1200px;
  margin: auto;
}
.domain-exp .q-item__section {
  padding: 0px !important; /* Remove extra padding in domain groups */
}
.eml-mng-div, .eml-card {
  min-height: 400px;
}
</style>
