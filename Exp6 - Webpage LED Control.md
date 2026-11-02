from http.server import HTTPServer, BaseHTTPRequestHandler
from gpiozero import LED
import time

led = LED(17)


class Server(BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path == "/blink":
            for i in range(5):
                led.on()
                time.sleep(0.3)
                led.off()
                time.sleep(0.3)

        html = """
        <html>
        <body>
            <h1>LED Control</h1>
            <button onclick="fetch('/blink')">Blink LED</button>
        </body>
        </html>
        """

        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(html.encode())


HTTPServer(("0.0.0.0", 8000), Server).serve_forever()
