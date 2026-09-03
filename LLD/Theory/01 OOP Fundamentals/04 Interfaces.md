6. Interface

Access modifiers control visibility within a class. But in larger applications, we also need a way to define behavior that different classes can provide in their own way.

That brings us to interfaces.

An interface defines a contract. It tells us what a class must do without deciding exactly how it should do it.

For example, imagine that our application needs to store files:

interface FileStorage {
    void save(String fileName, byte[] data);
}

This interface says that every file-storage implementation must provide a save method.
We can now create different implementations:
class LocalDiskStorage implements FileStorage {
    public void save(String fileName, byte[] data) {
        System.out.println("Saving " + fileName + " to local disk");
    }
}

class CloudStorage implements FileStorage {
    public void save(String fileName, byte[] data) {
        System.out.println("Uploading " + fileName + " to cloud storage");
    }
}


The rest of the application can depend on the interface instead of a specific storage system:


class BackupService {
    private FileStorage storage;

    BackupService(FileStorage storage) {
        this.storage = storage;
    }

    void backup(String fileName, byte[] data) {
        storage.save(fileName, data);
    }
}

Now BackupService can work with local storage, cloud storage, or any future implementation of FileStorage.

We can change the storage provider without rewriting the backup logic. That makes the code more flexible and easier to test.

