# Ceramic-
Build sovereign user profiles using Ceramic Self.ID
We will create a simple Next.js application, which uses Self.ID. It will allow users to login to the website using their wallet of choice, which will be linked to their 3ID. Then, we will allow the user to write some data to their decentralized profile, and be able to retrieve it from the Ceramic Network.

For verification of this level, we will ask you to enter your profile's StreamID at the end.

Let's get started by creating a new next app. Run the following command to create a new Next.js application inside a folder named ceramic-tutorial

npx create-next-app@latest ceramic-tutorial
and press Enter for all the question prompts. This should create the ceramic-tutorial folder and setup the next app inside it. It will also initialize a git repository you can push to GitHub after making changes.

Let's now install the Self.ID npm packages, and a dependent library, to get started. From inside the ceramic-tutorial folder, run the following in your terminal

npm install "@self.id/react" "@self.id/web" key-did-provider-ed25519
Let's also install the ethers and web3modal packages that we will be using to support wallet connections

npm install ethers web3modal
Open up the ceramic-tutorial folder in your text editor of choice, and let's get to coding.

The first thing we need to do is add Self.ID's Provider to the application. The SDK exposes a Provider component that needs to be added to the root of your web app. This initiatalizes the Self.ID instance, connects to the Ceramic Network, and makes Ceramic-related functionality available all across your app.

To do this, note in your pages folder Next.js automatically created a file called _app.js. This is the root of your web-app, and all other pages you create are rendered according to the configuration present in this file. By default, it does nothing special, and just renders your page directly. In our case, we want to wrap every component of ours with the Self.ID provider.

First, let's import the Provider from Self.ID. Add the following line at the top of _app.js

import { Provider } from "@self.id/react";
Next, change the MyApp function in that file to return the Component wrapped inside a Provider

function MyApp({ Component, pageProps }) {
  return (
    <Provider client={{ ceramic: "testnet-clay" }}>
      <Component {...pageProps} />;
    </Provider>
  );
}
We also specified a configuration option for the Provider. Specifically, we said that we want the Provider to connect to the Clay Test Network for Ceramic.

Let's make sure everything is working fine so far. Run the following in your terminal

npm run dev
Your website should be up and running at http://localhost:3000.

Lastly, before we start writing code for the actual application, replace the CSS in styles/Home.module.css with the following:

.main {
  min-height: 100vh;
}

.navbar {
  height: 10%;
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  padding-top: 1%;
  padding-bottom: 1%;
  padding-left: 2%;
  padding-right: 2%;
  background-color: orange;
}

.content {
  height: 80%;
  width: 100%;
  padding-left: 5%;
  padding-right: 5%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.flexCol {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.connection {
  margin-top: 2%;
}

.mt2 {
  margin-top: 2%;
}

.subtitle {
  font-size: 20px;
  font-weight: 400;
}

.title {
  font-size: 24px;
  font-weight: 500;
}

.button {
  background-color: azure;
  padding: 0.5%;
  border-radius: 10%;
  border: 0px;
  font-size: 16px;
  cursor: pointer;
}

.button:hover {
  background-color: beige;
}
Since this is not a CSS tutorial, we will not be diving deep into these CSS properties, though you should be able to understand what these properties mean if you would like to dive deeper.

Alright, open up pages/index.js and remove everything currently present inside the function Home() {...}. We will replace the contents of that function with our code.

We will start by first initializing Web3Modal related code, as connecting a wallet is the first step.

Let's first import Web3Provider from ethers.

import { Web3Provider } from "@ethersproject/providers";
Then import these react hooks from react and Web3Modal from web3modal.

import { useEffect, useRef, useState } from "react";
import Web3Modal from "web3modal";
As before, let's create a reference using the useRef react hook to a Web3Modal instance in your Home function, and create a helper function to get the Provider.

const web3ModalRef = useRef();

const getProvider = async () => {
  const provider = await web3ModalRef.current.connect();
  const wrappedProvider = new Web3Provider(provider);
  return wrappedProvider;
};
This function will prompt the user to connect their Ethereum wallet, if not already connected, and then return a Web3Provider. However, if you try running it right now, it will fail because we have not yet initialized web3Modal.

Before we initialize Web3Modal, we will setup a React Hook provided to us by the Self.ID SDK. Self.ID provides a hook called useViewerConnection which gives us an easy way to connect and disconnect to the Ceramic Network. Add the following import

import { useViewerConnection } from "@self.id/react";
And now in your Home function, do the following to initialize the hook

const [connection, connect, disconnect] = useViewerConnection();
Now that we have this, we can initialize Web3Modal. Add the following useEffect React Hook to your Home function.

useEffect(() => {
  if (connection.status !== "connected") {
    web3ModalRef.current = new Web3Modal({
      network: "goerli",
      providerOptions: {},
      disableInjectedProvider: false,
    });
  }
}, [connection.status]);
We have seen this code before many times. The only difference is the conditional here. We are checking that if the user has not yet been connected to Ceramic, we are going to initialize the web3Modal.

The last thing we need to be able to connect to the Ceramic Network is something called an EthereumAuthProvider. It is a class exported by the Self.ID SDK which takes an Ethereum provider and an address as an argument, and uses it to connect your Ethereum wallet to your 3ID.

To set that up, let us first import it

import { EthereumAuthProvider } from "@self.id/web";
and then add the following helper functions

const connectToSelfID = async () => {
  const ethereumAuthProvider = await getEthereumAuthProvider();
  connect(ethereumAuthProvider);
};

const getEthereumAuthProvider = async () => {
  const wrappedProvider = await getProvider();
  const signer = wrappedProvider.getSigner();
  const address = await signer.getAddress();
  return new EthereumAuthProvider(wrappedProvider.provider, address);
};
getEthereumAuthProvider creates an instance of the EthereumAuthProvider. You may be wondering why we are passing it wrappedProvider.provider instead of wrappedProvider directly. It's because ethers abstracts away the low level provider calls with helper functions so it's easier for developers to use, but since not everyone uses ethers.js, Self.ID maintains a generic interface to actual provider specification, instead of the ethers wrapped version. We can access the actual provider instance through the provider property on wrappedProvider. connectToSelfID takes this Ethereum Auth Provider, and calls the connect function that we got from the useViewerConnection hook which takes care of everything else for us.

Now, getting to the frontend a bit.

Add the following code to the end of your Home function

return (
  <div className={styles.main}>
    <div className={styles.navbar}>
      <span className={styles.title}>Ceramic Demo</span>
      {connection.status === "connected" ? (
        <span className={styles.subtitle}>Connected</span>
      ) : (
        <button
          onClick={connectToSelfID}
          className={styles.button}
          disabled={connection.status === "connecting"}
        >
          Connect
        </button>
      )}
    </div>

    <div className={styles.content}>
      <div className={styles.connection}>
        {connection.status === "connected" ? (
          <div>
            <span className={styles.subtitle}>
              Your 3ID is {connection.selfID.id}
            </span>
            <RecordSetter />
          </div>
        ) : (
          <span className={styles.subtitle}>
            Connect with your wallet to access your 3ID
          </span>
        )}
      </div>
    </div>
  </div>
);
For the explanation, here's what's happening. In the top right of the page, we conditionally render a button Connect that connects you to SelfID, or if you are already connected, it just says Connected. Then, in the main body of the page, if you are connected, we display your 3ID, and render a component called RecordSetter that we haven't yet created (we will soon). If you are not connected, we render some text asking you to connect your wallet first.

So far, we can connect to Ceramic Network through Self.ID, but what about actually storing and retrieving data on Ceramic?

We will consider the use case of building a decentralized profile on Ceramic. Thankfully, it is such a common use case that Self.ID comes with built in suport for creating and editing your profile. For the purposes of this tutorial, we will only set a Name to your 3ID and update it, but you can extend it to include all sorts of other properties like an avatar image, your social media links, a description, your age, gender, etc. For readability, we will divide this into a second React component. This is the RecordSetter component we used above.

Start by creating a new component in the same file. Outside your Home function, create a new function like this

function RecordSetter() {}
We are going to use another hook provided to us by Self.ID called useViewerRecord which allows storing and retrieving profile information on Ceramic Network. Let's first import it

import { useViewerRecord } from "@self.id/react";
Now, let's use this hook in the RecordSetter component.

const record = useViewerRecord("basicProfile");

const updateRecordName = async (name) => {
  await record.merge({
    name: name,
  });
};
We also created a helper function to update the name stored in our record (data on Ceramic). It takes a parameter name and updates the record.

Lastly, let's also create a state variable for name that you can type out to update your record

const [name, setName] = useState("");
For the frontend part, add the following at the end of your RecordSetter function

return (
  <div className={styles.content}>
    <div className={styles.mt2}>
      {record.content ? (
        <div className={styles.flexCol}>
          <span className={styles.subtitle}>Hello {record.content.name}!</span>

          <span>
            The above name was loaded from Ceramic Network. Try updating it
            below.
          </span>
        </div>
      ) : (
        <span>
          You do not have a profile record attached to your 3ID. Create a basic
          profile by setting a name below.
        </span>
      )}
    </div>

    <input
      type="text"
      placeholder="Name"
      value={name}
      onChange={(e) => setName(e.target.value)}
      className={styles.mt2}
    />
    <button onClick={() => updateRecordName(name)}>Update</button>
  </div>
);
This code basically renders a message Hi ${name} if you have set a name on your Ceramic profile record, otherwise it tells you that you do not have a profile set up yet and you can create one. You can create or update the profile by inputting text in the textbox and clicking the update button

We should be good to go at this point! Run

npm run dev
