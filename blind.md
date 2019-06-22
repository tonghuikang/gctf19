The site: https://bnv.web.ctfcompetition.com/

I tried to decode the braille image in the we, but nothing is suspicous. https://en.wikipedia.org/wiki/English_Braille

For this question, I understood what this script is doing. It encodes the alphabetical string. Some comments are added in the code.

```
function AjaxFormPost() {
  var datasend;
  var message = document.getElementById('message').value;
  message = message.toLowerCase();

  var blindvalues = [
    '10',    '120',   '140',    '1450',   '150',   '1240',  '12450',
    '1250',  '240',   '2450',   '130',    '1230',  '1340',  '13450',
    '1350',  '12340', '123450', '12350',  '2340',  '23450', '1360',
    '12360', '24560', '13460',  '134560', '13560',
  ];  // the API seem to ignore characters as encrypted

  var blindmap = new Map();
  var i;
  var message_new = '';

  for (i = 0; i < blindvalues.length; i++) {
    blindmap[i + 97] = blindvalues[i];  // 97 because the lowercase characters starts at 97
  }

  for (i = 0; i < message.length; i++) {
    message_new += blindmap[(message[i].charCodeAt(0))];
  }
  console.log(message_new);

  datasend = JSON.stringify({
    'message': message_new,
  });
  var url = '/api/search';
  xhr = new XMLHttpRequest();
  xhr.open('POST', url, true);
  xhr.setRequestHeader('Content-type', 'application/json');

  xhr.onreadystatechange =
      function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        console.log(xhr.getResponseHeader('Content-Type'));
        if (xhr.getResponseHeader('Content-Type') == "application/json; charset=utf-8") {
            try {
                var json = JSON.parse(xhr.responseText);
                document.getElementById('database-data').value = json['ValueSearch'];
            }
            catch(e) {;
                document.getElementById('database-data').value = e.message;
            }
        }
        else {
            document.getElementById('database-data').value = xhr.responseText;
        }
    }
}
      xhr.send(datasend);
}
```

I tried to use contractions in P(ar)is, but it just results "no result found".