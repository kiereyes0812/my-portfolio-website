<template>
	<div class="container mb-4" id="contact">
	<div class="row m-mt-5 pt-4">
		<div class="col-md-12 mb-4">
			<h2 class="text-center">Contact</h2>
		</div>
	</div>
	
	<div class="row">
		<div class="col-lg-6 d-none d-md-block">
			<div class="ratio ratio-4x3">
				<iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d373591.95705907204!2d140.91833085772421!3d42.98486313517186!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x5f0ad4755a973633%3A0x33937e9d4687bad5!2sSapporo%2C%20Hokkaido%2C%20Japan!5e0!3m2!1sen!2sph!4v1751483553938!5m2!1sen!2sph" width="500" height="300" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
			</div>
		</div>
		
		<div class="col-lg-6">
			<form>
				<div class="d-flex m-2">
			    	<input type="text" v-model="name" class="form-control" placeholder="Full name" aria-label="Full name">
				</div>
				<div class="d-flex m-2">
					<input type="email" v-model="email" class="form-control" id="exampleFormControlInput1" placeholder="Email">
				</div>
				<div class="d-flex d-inline-block m-2">
				  	<textarea v-model="message" class="form-control" id="exampleFormControlTextarea1" placeholder="Message" rows="5"></textarea>
				</div>
			  	<div class="text-right m-2">
			  		<button type="submit" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#confirmationModal">{{isLoading ? "Sending..." : "Submit"}}</button>
			  	</div>
			  	<div class="d-flex justify-content-end mt-2">
			  	    <div ref="recaptchaContainer"></div>
			  	</div>
			</form>
		</div>
	</div>
</div>
<!-- contact section end -->

<!-- Confirmation Modal -->
<div class="modal fade" id="confirmationModal" tabindex="-1" aria-labelledby="confirmationModalLabel" aria-hidden="true">
	<div class="modal-dialog modal-dialog-centered">
		<div class="modal-content">
			<div class="modal-header">
				<h5 class="modal-title" id="confirmationModalLabel">Message Sent</h5>
				<button
				type="button"
				class="btn-close"
				data-bs-dismiss="modal"
				aria-label="Close"
				></button>
			</div>
			<div class="modal-body">
				✅ Thank you! Your message has been sent successfully.
			</div>
			<div class="modal-footer">
				<button type="button" class="btn btn-primary" data-bs-dismiss="modal">
					OK
				</button>
			</div>

		</div>
	</div>
</div>
<!-- Confirmation Modal -->
</template>

<script setup>
	import { ref, onMounted, onBeforeUnmount } from "vue";

    import { Notyf } from "notyf";
    import 'notyf/notyf.min.css';

    const notyf = new Notyf();
	const WEB3FORMS_ACCESS_KEY = "162864f8-e192-40b6-97ef-9c66fc767257";
	const subject = "New message from Portfolio Contact Form";

	const name = ref("");
    const email = ref("");
    const message = ref("");
    const isLoading = ref(false);

    const submitForm = async () => {

        if(!recaptchaToken.value) {
            notyf.error('Please verify that you are not a robot.')
            return;
        }

        isLoading.value = true;
        try {
            const response = await fetch("https://api.web3forms.com/submit", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Accept: "application/json"
                },
                body: JSON.stringify({
                    access_key: WEB3FORMS_ACCESS_KEY,
                    subject: subject,
                    name: name.value,
                    email: email.value,
                    message: message.value
                }),
            });
            const result = await response.json();
            if (result.success){
                console.log(result);
                isLoading.value = false;
                notyf.success("Message Sent!")
            }
        } catch (error) {
            console.log(error)
            isLoading.value = false;
            notyf.error("Failed to send message");
        } finally {
            // Reset captcha after submit or error
            resetRecaptcha()
        }
    }

    const SITE_KEY = '6LfRBNwrAAAAAJu4-x0kS7CL03qi5wJyOekeCAPI'
    const recaptchaContainer = ref(null);
    const recaptchaWidgetId = ref(null);
    const recaptchaToken = ref('')

    function onRecaptchaSuccess(token){
        recaptchaToken.value = token;
    }

    function onRecaptchaExpired() {
        recaptchaToken.value = '';
    }

    function renderRecaptcha() {
        if(!window.grecaptcha) {
            console.error('reCAPTCHA not loaded');
        }

        recaptchaWidgetId.value = window.grecaptcha.render(
            recaptchaContainer.value, {
                sitekey: SITE_KEY,
                size: 'normal', //compact
                callback: onRecaptchaSuccess,
                'expired-callback': onRecaptchaExpired,

        });
    }

    function resetRecaptcha() {
        if (recaptchaWidgetId.value !== null) {
            window.grecaptcha.reset(recaptchaWidgetId.value)
            recaptchaToken.value = '';
        }
    }
    
    onMounted(() => {
        const interval = setInterval(() => {
            if(window.grecaptcha && window.grecaptcha.render) {
                renderRecaptcha();
                clearInterval(interval)
            }
        }, 100)

        onBeforeUnmount(() => {
            clearInterval(interval);
        })
    })


</script>
