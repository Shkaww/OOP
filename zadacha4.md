Решил делать через Декоратор, пускай он и менее гибкий зато не надо писать много классов
РЕализовал декоратор, создал интерфейс IEnhancedDatabaseSaver для чистоты кода и сам декоратор использую в DatabaseSaverClient

using Patterns.Ex04.ExternalLibs;

namespace Patterns.Ex04.SubEx_1
{
    public interface IEnhancedDatabaseSaver : IDatabaseSaver
    {
    }

    public class DatabaseSaverDecorator : IEnhancedDatabaseSaver
    {
        private readonly IDatabaseSaver _wrappedSaver;
        private readonly MailSender _mailSender;
        private readonly CacheUpdater _cacheUpdater;

        public DatabaseSaverDecorator(IDatabaseSaver saver, MailSender mailSender, CacheUpdater cacheUpdater)
        {
            _wrappedSaver = saver;
            _mailSender = mailSender;
            _cacheUpdater = cacheUpdater;
        }

        public void SaveData(object data)
        {
            _wrappedSaver.SaveData(data);
            _mailSender.Send("mail@example.com");
            _cacheUpdater.UpdateCache();
        }
    }

    public class DatabaseSaverClient
    {
        public void Main(bool b)
        {
            var databaseSaver = new DatabaseSaver();
            var mailSender = new MailSender();
            var cacheUpdater = new CacheUpdater();
            
            var decoratedSaver = new DatabaseSaverDecorator(databaseSaver, mailSender, cacheUpdater);
            DoSmth(decoratedSaver);
        }

        private void DoSmth(IDatabaseSaver saver)
        {
            saver.SaveData(null);
        }
    }
}