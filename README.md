# botframework-node-v3-receive-attachment

### Overview

When attachments are sent to the bot they can be found in the message activity [attachments](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) property.
This property exposes a list of [IAttachment](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html) representing each of the files sent. The attachment object contains information on the [contentType](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html#contenttype) of the file and a [contentUrl](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html#contenturl)  which is a reference to the location of the attachment's content. In order to access the actual attachment file you will need to download the address from the contentUrl property.
Check out the key code located in the [dialog handler](app.js#L26-L46) where the `message.attachments` property of the message activity is read to get the first attachment and download it.

> Note: The Skype and MS Teams attachment URLs are secured by JwtToken; you should set the JwtToken of your bot as the authorization header for the HTTP GET request your bot initiates to fetch content. Below is the sample code that temporarily works around this issue and set the JwtToken on the HTTP request. You should be careful when you send the bot's JwtToken to a third party server and should always make sure to send it to trusted parties.

````JavaScript
function (session) {

    var msg = session.message;
    if (msg.attachments.length) {

        // Message with attachment, proceed to download it.
        // Skype & MS Teams attachment URLs are secured by a JwtToken, so we need to pass the token from our bot.
        var attachment = msg.attachments[0];
        var fileDownload = checkRequiresToken(msg)
            ? requestWithToken(attachment.contentUrl)
            : request(attachment.contentUrl);

        fileDownload.then(
            function (response) {

                // Send reply with attachment type & size
                var reply = new builder.Message(session)
                    .text('Attachment of %s type and size of %s bytes received.', attachment.contentType, response.length);
                session.send(reply);

            }).catch(function (err) {
                console.log('Error downloading attachment:', { statusCode: err.statusCode, message: err.response.statusMessage });
            });
    }
}

// Helper methods

// Request file with Authentication Header
var requestWithToken = function (url) {
    return obtainToken().then(function (token) {
        return request({
            url: url,
            headers: {
                'Authorization': 'Bearer ' + token,
                'Content-Type': 'application/octet-stream'
            }
        });
    });
};

// Promise for obtaining JWT Token (requested once)
var obtainToken = Promise.promisify(connector.getAccessToken.bind(connector));

var checkRequiresToken = function (message) {
    return message.source === 'skype' || message.source === 'msteams';
};
````

### More Information

To get more information about how to get started in Bot Builder for Node.js and Attachments please review the following resources:
* [Bot Builder for Node.js Reference](https://docs.microsoft.com/en-us/bot-framework/nodejs/)
* [Message.attachments](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments)
* [Attachment](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html)

