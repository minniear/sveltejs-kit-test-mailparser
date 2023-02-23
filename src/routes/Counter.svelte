<script>
	import { spring } from 'svelte/motion';
	import{ fs } from 'file-system'
	import { MailParser } from 'mailparser'

// const mailpath = process.argv[2];

let parser = new MailParser();
let input = fs.createReadStream('/home/projects/sveltejs-kit-template-default-cqhn5t/src/nodemailer.eml')
let mailobj = {
	attachments: [],
    text: {}
};
	
input.pipe(parser);
const parsing = new Promise((resolve) => {


parser.on('headers', headers => {
    let headerObj = {};
    for (let [k, v] of headers) {
        // We don’t escape the key '__proto__'
        // which can cause problems on older engines
        headerObj[k] = v;
    }

    mailobj.headers = headerObj;
});

parser.on('data', data => {
    if (data.type === 'attachment') {
        mailobj.attachments.push(data);
        data.content.on('readable', () => data.content.read());
        data.content.on('end', () => data.release());
    } else {
        mailobj.text = data;
    }
});

parser.on('content-type', content => {
	console.log(content)

})

parser.on('end', () => {
    console.log(mailobj.headers['content-type'].params);
		resolve();
});
})
</script>

<div class="counter">
	{#await parsing}
	Loading...
	{:then mailobj}
	{mailobj.text}
	{:catch error}
	<p style="color: red">{error.message}</p>
{/await}

</div>
