If you want to use google oauth scopes like gmail.read or gmail.write, starting from feb 2019 google does a manual verification.

Checklist for Verification.

- [ ] Your application should serve over HTTPS.
- [ ] Create a privacy page, clearly disclose the manner in which your application accesses, uses, stores, or shares Google user data. Your use of Google user data must be limited to the practices explicitly disclosed in your published privacy policy.
- [ ] Your Privacy policy must be visible to users, accessible from the app’s home page, hosted within the domain of your website, and linked to the OAuth consent screen on the Google API Console.
- [ ] Go to the Search Console to complete the domain verification process. The account you use must be either a Project Owner or a Project Editor of the Google Cloud Project.

More on [Google User Data Policy](https://developers.google.com/terms/api-services-user-data-policy)
