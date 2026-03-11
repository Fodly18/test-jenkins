node {
    // Menarik kode dari GitHub (Fodly18/test-jenkins)
    checkout scm

    // 1. Build Stage
    stage("Build") {
        // Gunakan image PHP yang lebih baru (8.2) agar sesuai dengan Laravel modern
        docker.image('composer:latest').inside('-u root') {
            sh 'composer install --no-interaction --prefer-dist'
        }
    }

    // 2. Testing Stage (Penting untuk tugas akhir)
    stage("Testing") {
        docker.image('php:8.2-cli').inside('-u root') {
            sh 'cp .env.example .env'
            sh 'php artisan key:generate'
            // Menjalankan unit test asli Laravel
            sh 'php artisan test'
        }
    }

    // 3. Deploy Stage
    stage("Deploy") {
        // Pastikan plugin SSH Agent sudah kamu instal di Jenkins tadi
        docker.image('instrumentisto/rsync-ssh').inside('-u root') {
            // 'ssh-prod' adalah ID Kredensial yang harus kamu buat di Jenkins
            sshagent (credentials: ['ssh-prod']) {
                sh 'mkdir -p ~/.ssh'
                // Pastikan variabel PROD_HOST sudah didaftarkan di Jenkins Environment
                sh "ssh-keyscan -H ${env.PROD_HOST} >> ~/.ssh/known_hosts"
                // Perhatikan path ./ (root) jika file Laravel tidak di dalam subfolder
                sh "rsync -rav --delete ./ ubuntu@${env.PROD_HOST}:/home/ubuntu/laravel-app/ --exclude=.env --exclude=storage --exclude=.git --exclude=vendor"
            }
        }
    }
}
