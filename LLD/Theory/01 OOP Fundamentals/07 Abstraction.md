9. Abstraction
Encapsulation controls access to internal data. But sometimes, we also want to hide the internal complexity of an operation or method.

That brings us to abstraction.

Think about a video player library and its clients. To play a video, the caller should not need to understand networking, buffering, codecs, or frame decoding. It should only need to call a simple method:

class VideoPlayer {

    public void play(String fileName) {
        connect(fileName);
        buffer();
        decodeFrames();
        render();
    }

    private void connect(String fileName) { /* networking */ }
    private void buffer() { /* buffering strategy */ }
    private void decodeFrames() { /* codecs and decoding */ }
    private void render() { /* frame rendering */ }
}

// The caller only needs this:
VideoPlayer player = new VideoPlayer();
player.play("movie.mp4");


The caller knows what the play method does, but it does not need to know every detail of how it works. That complexity is hidden behind a simple public API.

This is abstraction. It reduces the amount of information developers need to understand at one time.

Interfaces and abstract classes are two common ways to create abstractions, but abstraction is a broader idea. Any well-designed class or method can provide abstraction by exposing a simple API and hiding unnecessary details.


Encapsulation vs Abstraction

A simple way to distinguish the two:

Encapsulation controls who can access or modify something.
Abstraction controls which details the caller needs to see.

