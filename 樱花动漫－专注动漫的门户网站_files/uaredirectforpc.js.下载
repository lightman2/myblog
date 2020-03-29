(function () {
var ua = window.navigator.userAgent,
agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPod' , 'iPad'],
isPC = true;
for (var i = 0, len = agents.length; i < len; i++) {
if (ua.indexOf(agents[i]) > 0) {
isPC = false;
break;
}
}
if (isPC == false) {
var href = window.location.href.replace('www', 'm');
window.location.href = href;
}
})();