
#Bitmarkets

Bitmarkets is a protocol and client used to implement a p2p online marketplace client using Bitmessage for communications and Bitcoin for payments. It also supports a decentralized review system.

##Protocol Specification

###Message format

All messages are sent as JSON dictionaries in a bitmessage set to use the "SIMPLE" encoding with the JSON in the body section of the message. The subject field is a human readable comment ignored by the protocol but convenient for debugging in the format: "Bitmarkets/[message type]/[message hash]"


###Message objects

TimeSpace restrictions object

	{
		responsePeriod: dt,
		deliveryTimeRange: [t1, t2],
		location: {
			// address with only restrictions specified, e.g. zip code or geospacial range
		}
	}


###Message types

These are present in the order in which they would typically be sent.

#### ask // sent by seller
	{
		service: "bitmarket",
		msg: "ask",
		restrictions: {},
		category: ["pathComponent0", "pathComponent1", ...],
		title: "",
		description: "",
		amount: { value: 0.0 , unit: "" },
		ask: { amount: 0.0, unit: "BTC" }
	}

#### bid // sent by buyer
	{
		service: "bitmarket",
		msg: "bid",
		askRefSig: "",
		restrictions: {},
		bid: { amount: 0.0, unit: "BTC" },
	}

#### bidReject // sent by seller
{
		service: "bitmarket",
		msg: "bidReject",
		bidRefSig: ""
}

#### bidAccept // sent by seller
	{
		service: "bitmarket",
		msg: "bidAccept",
		bidRefSig: "",
		restrictions: {}
		hashOfSellerSecret: ""
	}

// buyer constructs multisigAddress and sends with these key sigs required 
// sellerMSPubKey - generated by seller for this tx
// buyerMSPubKey - generated by buyer for this tx
// never send more than you're willing to loose unless!
// weigh amount your will to risk by review history of seller

#### bidEscrowed // sent by buyer
	{
		service: "bitmarket",
		bidAcceptSig: "",
		msg: "bidEscrowed",
		payload: "[buyer signed tx data]",
	}

// seller verifies escrowTxHash script constructed properly
// sends to bitcoin network

#### escrowPublished // sent by seller
	{
		service: "bitmarket",
		bidEscrowedMsgId: "",
		msg: "escrowPublished",
	}

// buyer verifies that escrow tx is in bitcoin network

#### deliveryRequest [direct message] // sent by buyer
	{
		service: "bitmarket",
		bidAcceptSig: "",
		escrowTxHash: "",
		deliveryAddress: {}, // optional		
		address: {},
	}

#### deliveryStatus [direct message] // sent by seller
	{
		service: "bitmarket",
		bidEscrowedSig: "",
		eta: "",
		status: "", 
		note: ""
	}
 
#### payment [direct message] // sent by buyer on delivery
	{
		service: "bitmarket",
		sellerMSPrivateKey: "",
	}
 
#### refundRequest // sent by buyer on failure to deliver
	{
		service: "bitmarket",
		buyerMSPrivateKey: "",
	}

#### refund // sent by seller 
	{
		service: "bitmarket",
		bidEscrowedSig: "",
		privateKey: "", // private key  generated K1 in escrow
	}

// client can verify bidEscrowed and whether payment was sent
// to sellerPaymentAddress (completed) or another address (

#### review
	{
		service: "bitmarket",
		bidAcceptRef: "",
		status: "", // delivered, not delivered, delivered but unacceptable
		rating: 0-5,
		content: "",
	}
