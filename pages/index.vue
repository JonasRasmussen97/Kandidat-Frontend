<template>
  <div class="container">
    <!-- Issuer Section </-->
    <div class="row">
      <div class="col">
        <b>Select Issuer Organisation</b>
        <b-form-select v-model="selectedIssuer" :options="issuerOptions"></b-form-select>
      </div>
    </div>
    <!-- Arrow </-->
    <div class="row pt-4" v-if="selectedIssuer !== null">
      <div class="col">
         <b-img center src="../static/arrow.png" width="60rem"></b-img>
      </div>
    </div>
   <!-- Intermediate Section </-->
    <div class="row pt-2" v-if="selectedIssuer !== null">
      <div class="col">
        <b>Select Intermediate Organisation</b>
        <b-form-select v-model="selectedIntermediate" :options="intermediateOptions"></b-form-select>
      </div>
    </div>
    <!-- Arrow </-->
      <div class="row pt-4" v-if="selectedIntermediate !== null">
      <div class="col">
         <b-img center src="../static/arrow.png" width="60rem"></b-img>
      </div>
    </div>
    <!-- Buttons </-->
     <div class="row pt-4 text-center" v-if="selectedIntermediate !== null">
      <div class="col">
         <b-button @click="issueCertificate(selectedIssuer, selectedIntermediate, 'Intermediate')" variant="primary">Issue Certificate from <b>{{selectedIssuer}}</b> to <b>{{selectedIntermediate}}</b></b-button>
      </div>
    </div>

    <!-- MSP </-->
    <h2 v-if="certificates.length > 0" class="p-4 text-center"><b>Membership Service Provider (MSP)</b></h2>
    <div class="row pt-2" v-if="certificates.length > 0">
      <div class="col">
       <b>Select Certificate</b>
         <b-form-select v-model="selectedCertificate" :options="certificates"></b-form-select>
      </div>
    </div>

  <!-- MSP Buttons </-->
    <div class="row pt-2 text-center" v-if="certificates.length > 0">
    <div class="col">
        <b-button @click="revokeCertificate(selectedCertificate)" variant="danger">Revoke Certificate</b-button>
        <b-button variant="success">Reenroll Certificate</b-button>
    </div>
  </div>

  <!-- MSP Text </-->
    <div class="row" v-if="certificates.length > 0">
      <div class="col text-center">
      <p><i>In order to <b>Revoke</b> or <b>Reenroll</b> a certificate it requires that you operate with a certificate (Identity) that has the required permissions.</i></p>
      <i>The MSP has a reference to one or more Certificate Revocation List(s) (CRL) which contains a list of all of the certificates that have been revoked.</i>
      </div>
    </div>

  <!-- Chain of Trust </-->
    <div class="row pt-2" v-if="certificateAffiliations.length > 0">
      <div class="col text-center">
        <h2><b>Chain of Trust</b></h2>
      </div>
    </div>

     <div class="row pt-2">
      <div class="col text-center">
      <div v-for="(affiliation, index) in certificateAffiliations" :key=index>
        <p v-if="affiliation.to !== 'Client'"><b-button @click="issueCertificate(affiliation.to, 'Client', 'Client')" size="sm">Issue certificate from <b>{{affiliation.to}}</b> to <b>Client</b></b-button></p>
          <b>{{affiliation.from}}</b> <b-img class="pl-2" src="../static/arrow-right.png" width="50rem"></b-img> <b>{{affiliation.to}}</b>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
   data() {
    return {
      selectedIssuer: null,
      selectedIntermediate:null,
      selectedCertificate: null,
      issuerOptions: [{value: "Energinet", text: "Energinet"}],
      intermediateOptions: [{value: "EnergiFyn", text: "Energi Fyn"}, {value: "GulStrom", text: "Gul Strøm"}, {value: "Norlys", text: "Norlys"}],
      certificates: [],
      revokedCertificates: [],
      certificateAffiliations: [],
    }
  },
  methods: {
    // Type here defines e.g., if it is an intermediate cert or not.
    issueCertificate(issuer, holder, type) {
      let cert = (Math.random() + 1).toString(36);
      // If intermediate, then we add the certificate holder to the issuer list.
      // We ensure to only add the name once, so we don't add the same name multiple times.
      if(type == "Intermediate" && !this.issuerOptions.some(e => e.value === holder)) {
          this.issuerOptions.push({cert: cert, value: holder, text: holder});
      }
      // We add the certificate to the cert list.
      this.certificates.push(cert);
      // We create the affiliation 
      this.certificateAffiliations.push({cert: cert, from: issuer, to: holder, type: type})
      

    },
    revokeCertificate(cert) {
      // Remove the certificate from the certificate list.
      let index1 = this.certificates.indexOf(cert);
      this.certificates.splice(index1, 1);
      
      // Remove the affiliation
      let index2 = this.certificateAffiliations.findIndex(function(aff) {
        return aff.cert === cert;
      })
      this.certificateAffiliations.splice(index2, 1);

      // Remove the holder from issuer list in case they were an intermediate.
      let index3 = this.issuerOptions.findIndex(function(iss) {
        return iss.cert === cert;
      })
      this.issuerOptions.splice(1, index3);

    },
    affiliationExists(issuer) {
      return this.certificateAffiliations.findIndex(function(aff) {
        return aff.from == issuer;
      })
    }

  }
}
</script>
