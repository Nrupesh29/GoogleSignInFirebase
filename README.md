# Google SignIn Firebase

1. Go to https://developers.google.com/android/guides/client-auth and generate SHA-1

2. Create Firebase Project and add Android App using the app package name and SHA-1.

3. Go to Authentication tab and enable Google in Sign In options.

4. Go to Project Settings and download `google-services.json` file and add to the `app/` folder.

5. Add following line to Project Level build.gradle

    ```gradle
    buildscript {
      dependencies {
        // Add this line
        classpath 'com.google.gms:google-services:3.0.0'
      }
    }
    ```

6. Add following lines to App Level build.gradle

      ```gradle
        dependencies {
            ...
            // Add the below two lines
            compile 'com.google.firebase:firebase-auth:10.0.1'
            compile 'com.google.android.gms:play-services-auth:10.0.1'
            ...
        }
        
        // Add this line to bottom of file
        apply plugin: 'com.google.gms.google-services'
      ```
      
7. Add following code to LoginActivity.java file.

      ```java

      public class LoginActivity extends AppCompatActivity {

          private FirebaseAuth mAuth;
          private FirebaseAuth.AuthStateListener mAuthListener;
          private static final String TAG = "LoginActivity";
          private static final int RC_SIGN_IN = 9001;
          private GoogleApiClient mGoogleApiClient;
          private ProgressDialog loading;

          @Override
          protected void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
              StrictMode.setThreadPolicy(policy);
              setContentView(R.layout.activity_login);

              GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                      .requestIdToken(getString(R.string.default_web_client_id))
                      .requestEmail()
                      .build();

              mGoogleApiClient = new GoogleApiClient.Builder(this)
                      .enableAutoManage(this, new GoogleApiClient.OnConnectionFailedListener() {
                          @Override
                          public void onConnectionFailed(@NonNull ConnectionResult connectionResult) {

                          }
                      })
                      .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
                      .build();

              Button signInButton = (Button) findViewById(R.id.signInButton);
              signInButton.setOnClickListener(new View.OnClickListener() {
                  @Override
                  public void onClick(View view) {
                      loading = ProgressDialog.show(LoginActivity.this, null, "Signing In...", true, true);
                      loading.setCancelable(false);
                      signIn();
                  }
              });

              mAuth = FirebaseAuth.getInstance();

              mAuthListener = new FirebaseAuth.AuthStateListener() {
                    @Override
                    public void onAuthStateChanged(@NonNull FirebaseAuth firebaseAuth) {
                        FirebaseUser user = firebaseAuth.getCurrentUser();
                        if (user != null) {
                            // User is signed in

                            startActivity(new Intent(LoginActivity.this, MainActivity.class));
                            if (loading.isShowing())
                                loading.dismiss();
                            finish();

                            Log.d(TAG, "onAuthStateChanged:signed_in:" + user.getUid());
                        } else {
                            // User is signed out
                            Log.d(TAG, "onAuthStateChanged:signed_out");
                        }
                        // ...
                    }
                };
            }

            @Override
            public void onStart() {
                super.onStart();
                mAuth.addAuthStateListener(mAuthListener);
            }

            @Override
            public void onStop() {
                super.onStop();
                if (mAuthListener != null) {
                    mAuth.removeAuthStateListener(mAuthListener);
                }
            }

            private void signIn() {
                Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
                startActivityForResult(signInIntent, RC_SIGN_IN);
            }

            @Override
            public void onActivityResult(int requestCode, int resultCode, Intent data) {
                super.onActivityResult(requestCode, resultCode, data);

                // Result returned from launching the Intent from GoogleSignInApi.getSignInIntent(...);
                if (requestCode == RC_SIGN_IN) {
                    GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
                    if (result.isSuccess()) {
                        // Google Sign In was successful, authenticate with Firebase
                      GoogleSignInAccount account = result.getSignInAccount();
                      firebaseAuthWithGoogle(account);
                  } else {
                      // Google Sign In failed, update UI appropriately
                      // ...
                      if (loading.isShowing())
                          loading.dismiss();
                  }
              }
          }

          private void firebaseAuthWithGoogle(GoogleSignInAccount acct) {
              Log.d(TAG, "firebaseAuthWithGoogle:" + acct.getId());

              AuthCredential credential = GoogleAuthProvider.getCredential(acct.getIdToken(), null);
              mAuth.signInWithCredential(credential)
                      .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                          @Override
                          public void onComplete(@NonNull Task<AuthResult> task) {
                              Log.d(TAG, "signInWithCredential:onComplete:" + task.isSuccessful());

                              // If sign in fails, display a message to the user. If sign in succeeds
                              // the auth state listener will be notified and logic to handle the
                              // signed in user can be handled in the listener.
                              if (!task.isSuccessful()) {
                                  if (loading.isShowing())
                                      loading.dismiss();
                                  Log.w(TAG, "signInWithCredential", task.getException());
                                  Toast.makeText(LoginActivity.this, "Authentication failed.",
                                          Toast.LENGTH_SHORT).show();
                              }
                              // ...
                          }
                      });
          }
      }

      ```
